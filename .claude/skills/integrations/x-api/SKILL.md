---
name: x-api
description: Integrate with X (Twitter) API to post tweets, threads, read timelines, search, and track analytics.
---

# X API

Programmatic interaction with X (Twitter) for posting, reading, searching, and analytics via REST API and OAuth.

---

## 1. Authentication

Use **OAuth 2.0 Bearer Token** for read-heavy operations (search, public data). Use **OAuth 1.0a** for write operations (posting, DMs, account management).

```bash
# Bearer token (read)
export X_BEARER_TOKEN="your-bearer-token"

# OAuth 1.0a (write)
export X_API_KEY="..."
export X_API_SECRET="..."
export X_ACCESS_TOKEN="..."
export X_ACCESS_SECRET="..."
```

```python
from requests_oauthlib import OAuth1Session
import os

oauth = OAuth1Session(
    os.environ["X_API_KEY"],
    client_secret=os.environ["X_API_SECRET"],
    resource_owner_key=os.environ["X_ACCESS_TOKEN"],
    resource_owner_secret=os.environ["X_ACCESS_SECRET"],
)
```

**Rule:** Never hardcode tokens. Use environment variables or `.env` files. Add `.env` to `.gitignore`. Use read-only tokens when write access is not needed.

---

## 2. Core Operations

Post a single tweet:
```python
resp = oauth.post("https://api.x.com/2/tweets", json={"text": "Hello from Claude Code"})
resp.raise_for_status()
tweet_id = resp.json()["data"]["id"]
```

Post a thread:
```python
def post_thread(oauth, tweets: list[str]) -> list[str]:
    ids, reply_to = [], None
    for text in tweets:
        payload = {"text": text}
        if reply_to:
            payload["reply"] = {"in_reply_to_tweet_id": reply_to}
        resp = oauth.post("https://api.x.com/2/tweets", json=payload)
        reply_to = resp.json()["data"]["id"]
        ids.append(reply_to)
    return ids
```

Search recent tweets:
```python
import requests
headers = {"Authorization": f"Bearer {os.environ['X_BEARER_TOKEN']}"}
resp = requests.get(
    "https://api.x.com/2/tweets/search/recent",
    headers=headers,
    params={"query": "from:user -is:retweet", "max_results": 10, "tweet.fields": "public_metrics,created_at"},
)
```

Upload media then post:
```python
media_resp = oauth.post("https://upload.twitter.com/1.1/media/upload.json", files={"media": open("image.png", "rb")})
media_id = media_resp.json()["media_id_string"]
oauth.post("https://api.x.com/2/tweets", json={"text": "Caption", "media": {"media_ids": [media_id]}})
```

---

## 3. Rate Limit Handling

X API rate limits vary by endpoint, auth method, and account tier — always check headers at runtime.

```python
import time

remaining = int(resp.headers.get("x-rate-limit-remaining", 0))
if remaining < 5:
    reset = int(resp.headers.get("x-rate-limit-reset", 0))
    wait = max(0, reset - int(time.time()))
    print(f"Rate limit approaching. Resets in {wait}s")
```

**Rule:** Never hardcode rate limit assumptions. Read `x-rate-limit-remaining` and `x-rate-limit-reset` headers and back off automatically.

---

## 4. Error Handling

```python
resp = oauth.post("https://api.x.com/2/tweets", json={"text": content})
if resp.status_code == 201:
    return resp.json()["data"]["id"]
elif resp.status_code == 429:
    raise Exception(f"Rate limited. Resets at {resp.headers['x-rate-limit-reset']}")
elif resp.status_code == 403:
    raise Exception(f"Forbidden: {resp.json().get('detail', 'check permissions')}")
else:
    raise Exception(f"X API error {resp.status_code}: {resp.text}")
```

---

## 5. Content Engine Integration

Use the `content-engine` skill to generate platform-native content, then post via X API:
1. Generate content with content-engine (X platform format, 280-char limit for single tweets)
2. Validate length
3. Post using OAuth 1.0a patterns above
4. Track engagement via `public_metrics` on the tweet
