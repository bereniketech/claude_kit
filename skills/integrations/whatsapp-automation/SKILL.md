---
name: whatsapp-automation
description: Automate WhatsApp messaging via Meta's WhatsApp Business Cloud API — send messages, templates, media, and handle inbound webhooks.
---

# WhatsApp Automation

Send and receive WhatsApp messages programmatically using Meta's WhatsApp Business Cloud API (v19.0+).

---

## 1. Credentials Setup

```bash
export WHATSAPP_TOKEN="your-permanent-system-user-token"
export WHATSAPP_PHONE_ID="your-phone-number-id"        # from Meta Developer console
export WHATSAPP_BUSINESS_ID="your-waba-id"
export WHATSAPP_WEBHOOK_SECRET="your-verify-token"
```

**Rule:** Never hardcode tokens. Use a system user token (not a temporary user token) for production — it does not expire.

---

## 2. Send Messages

Base URL: `https://graph.facebook.com/v19.0/{PHONE_ID}/messages`

**Text message:**
```python
import httpx, os

def send_text(to: str, body: str) -> dict:
    return httpx.post(
        f"https://graph.facebook.com/v19.0/{os.environ['WHATSAPP_PHONE_ID']}/messages",
        headers={"Authorization": f"Bearer {os.environ['WHATSAPP_TOKEN']}"},
        json={"messaging_product": "whatsapp", "to": to, "type": "text", "text": {"body": body}},
    ).raise_for_status().json()
```

**Template message** (required for first-contact or 24h+ window):
```python
def send_template(to: str, template: str, lang: str = "en_US", components: list = None) -> dict:
    payload = {
        "messaging_product": "whatsapp",
        "to": to,
        "type": "template",
        "template": {"name": template, "language": {"code": lang}},
    }
    if components:
        payload["template"]["components"] = components
    return httpx.post(
        f"https://graph.facebook.com/v19.0/{os.environ['WHATSAPP_PHONE_ID']}/messages",
        headers={"Authorization": f"Bearer {os.environ['WHATSAPP_TOKEN']}"},
        json=payload,
    ).raise_for_status().json()
```

**Media message** (image/document/audio/video):
```python
def send_media(to: str, media_type: str, url: str, caption: str = "") -> dict:
    return httpx.post(
        f"https://graph.facebook.com/v19.0/{os.environ['WHATSAPP_PHONE_ID']}/messages",
        headers={"Authorization": f"Bearer {os.environ['WHATSAPP_TOKEN']}"},
        json={
            "messaging_product": "whatsapp",
            "to": to,
            "type": media_type,
            media_type: {"link": url, "caption": caption},
        },
    ).raise_for_status().json()
```

**Rule:** You can only send free-form text within a 24-hour customer-initiated window. Outside that window, use an approved template.

---

## 3. Receive Messages (Webhook)

```python
from fastapi import FastAPI, Request, Response
import hmac, hashlib, os

app = FastAPI()

@app.get("/webhook")
def verify(hub_mode: str, hub_verify_token: str, hub_challenge: str):
    if hub_mode == "subscribe" and hub_verify_token == os.environ["WHATSAPP_WEBHOOK_SECRET"]:
        return Response(content=hub_challenge)
    return Response(status_code=403)

@app.post("/webhook")
async def receive(request: Request):
    payload = await request.body()
    sig = request.headers.get("x-hub-signature-256", "")
    expected = "sha256=" + hmac.new(
        os.environ["WHATSAPP_TOKEN"].encode(), payload, hashlib.sha256
    ).hexdigest()
    if not hmac.compare_digest(sig, expected):
        return Response(status_code=401)

    data = await request.json()
    for entry in data.get("entry", []):
        for change in entry.get("changes", []):
            messages = change.get("value", {}).get("messages", [])
            for msg in messages:
                handle_message(msg)
    return {"status": "ok"}

def handle_message(msg: dict):
    msg_type = msg["type"]        # "text", "image", "audio", "document", etc.
    sender   = msg["from"]        # phone number
    if msg_type == "text":
        text = msg["text"]["body"]
        # reply logic here
```

**Rule:** Always verify the `x-hub-signature-256` header before processing webhook payloads.

---

## 4. Mark Messages as Read

```python
def mark_read(message_id: str):
    httpx.post(
        f"https://graph.facebook.com/v19.0/{os.environ['WHATSAPP_PHONE_ID']}/messages",
        headers={"Authorization": f"Bearer {os.environ['WHATSAPP_TOKEN']}"},
        json={"messaging_product": "whatsapp", "status": "read", "message_id": message_id},
    )
```

---

## 5. Error Handling & Rate Limits

| HTTP code | Meaning | Action |
|-----------|---------|--------|
| 400 | Bad request / invalid parameter | Fix payload |
| 429 | Rate limited | Backoff with `Retry-After` header |
| 131047 | Message outside 24h window | Switch to approved template |
| 131026 | Phone number not on WhatsApp | Skip or flag |

```python
resp = httpx.post(url, headers=headers, json=payload)
if resp.status_code == 429:
    retry_after = int(resp.headers.get("Retry-After", 60))
    raise RateLimitError(f"Rate limited. Retry in {retry_after}s")
if not resp.is_success:
    err = resp.json().get("error", {})
    raise Exception(f"WA API {err.get('code')}: {err.get('message')}")
```

**Rule:** Log `error.code` and `error.error_subcode` for every failure — subcode 131047 is the most common production error and requires a template fallback, not a retry.

---

## 6. Template Management

Approved templates live in Meta Business Manager. Components:

```python
# Template with variable body + button
components = [
    {"type": "body", "parameters": [{"type": "text", "text": "John"}]},
    {"type": "button", "sub_type": "url", "index": "0",
     "parameters": [{"type": "text", "text": "order-123"}]},
]
send_template(to="+1234567890", template="order_confirmation", components=components)
```

**Rule:** Submit templates for approval before launch — approval takes minutes to days. Build with approved templates first; free-form replies can be added after the conversation window opens.
