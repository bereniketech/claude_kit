---
name: multi-model-model-agnostic-platforms
description: Work effectively across multiple AI models and model-agnostic platforms, selecting the right model for each task. Use when the user is building systems that use multiple AI models, comparing model capabilities, or working in multi-model environments. Synthesizes best practices from VSCode Agent, Augment Code, Amp, Traycer AI, and Cursor.
---

# Multi-Model / Model-Agnostic Platform Skill

You are operating in a multi-model environment. Help users build, maintain, and optimize systems that span multiple AI models and providers.

---

## 1. Model Selection Criteria

Choose the right model for each task by evaluating:

- **Task complexity**: Use large frontier models (Claude Sonnet/Opus, GPT-4o, Gemini Ultra) for complex reasoning, multi-step planning, and code generation. Use smaller models (Claude Haiku, GPT-4o mini, Gemini Flash) for classification, summarization, or high-volume triage.
- **Modality requirements**: Match the model to required input/output modalities — text, code, vision, audio, structured data, or tool use.
- **Context window needs**: Use models with larger context windows (200K+ tokens) for full codebase ingestion, long document analysis, or multi-turn conversations with heavy history.
- **Cost-per-token budget**: Estimate token consumption upfront. For high-volume pipelines, model selection is a budget decision as much as a capability decision.
- **Latency sensitivity**: Real-time user-facing tasks require low-latency models. Offline batch pipelines can use larger, slower, cheaper models.

**Decision rule**: Default to the smallest model that satisfies the task requirements. Escalate to a larger model only when the smaller model demonstrably fails.

**Model routing by task complexity (Python example):**
```python
MODEL_SONNET = "claude-sonnet-4-6"
MODEL_HAIKU = "claude-haiku-4-5-20251001"

def select_model(text_length: int, item_count: int, force_model: str | None = None) -> str:
    if force_model is not None:
        return force_model
    if text_length >= 10_000 or item_count >= 30:
        return MODEL_SONNET  # Complex task
    return MODEL_HAIKU        # Simple task (3-4x cheaper)
```

---

## 2. Cost-Aware Pipeline Patterns

**Immutable cost tracking** — track cumulative spend with frozen dataclasses:
```python
from dataclasses import dataclass

@dataclass(frozen=True, slots=True)
class CostTracker:
    budget_limit: float = 1.00
    records: tuple = ()

    def add(self, record) -> "CostTracker":
        return CostTracker(budget_limit=self.budget_limit, records=(*self.records, record))

    @property
    def total_cost(self) -> float:
        return sum(r.cost_usd for r in self.records)

    @property
    def over_budget(self) -> bool:
        return self.total_cost > self.budget_limit
```

**Narrow retry logic** — retry only on transient errors, fail fast on authentication or bad request errors:
```python
from anthropic import APIConnectionError, InternalServerError, RateLimitError

_RETRYABLE = (APIConnectionError, RateLimitError, InternalServerError)

def call_with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except _RETRYABLE:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)  # Exponential backoff
    # AuthenticationError, BadRequestError → raise immediately
```

**Prompt caching** — cache long system prompts to avoid resending on every request:
```python
messages = [{
    "role": "user",
    "content": [
        {"type": "text", "text": system_prompt, "cache_control": {"type": "ephemeral"}},
        {"type": "text", "text": user_input},
    ],
}]
```

**Pricing reference (2025-2026):**

| Model | Input ($/1M tokens) | Output ($/1M tokens) |
|-------|---------------------|----------------------|
| Haiku 4.5 | $0.80 | $4.00 |
| Sonnet 4.6 | $3.00 | $15.00 |
| Opus 4.5 | $15.00 | $75.00 |

---

## 3. Prompt Portability Across Models

Write prompts that work reliably across multiple models without per-model rewrites:

- **Avoid model-specific syntax**: Do not rely on proprietary prompt features. Use neutral Markdown for structure.
- **Explicit role and context framing**: Always open with a clear role statement and task description.
- **Structured output specification**: Define expected output format in the prompt body itself — do not rely on model default behaviors.
- **Separate system and user content**: Keep system-level instructions in the system prompt; task-specific content in the user turn. This is portable across all major APIs (Anthropic, OpenAI, Google, Mistral).
- **Use delimiters consistently**: Wrap distinct content sections in consistent delimiters (XML tags, triple-backtick blocks, or `---` separators).
- **Test prompt portability**: After writing a prompt, run it against at least two models. If outputs diverge significantly, the prompt is under-specified.

---

## 4. Handling Model-Specific Capabilities and Limitations

- **Tool/function calling schemas**: OpenAI, Anthropic, and Google each have different schemas. Abstract tool definitions into a canonical format and write per-provider adapters.
- **Context window utilization**: Monitor output quality as context grows; implement context pruning or summarization before hitting the limit.
- **Streaming support**: Implement a streaming abstraction with a non-streaming fallback.
- **Maximum output tokens**: Chunk long-output tasks into multiple sequential calls if output may exceed the model's generation limit.
- **Refusal behavior**: Each model has different content policies. Define the scope of your agent clearly in the system prompt so refusals are predictable.

---

## 5. Fallback Strategies When a Model Fails

Build resilient multi-model pipelines that degrade gracefully:

- **Define a model priority chain**: For every task type, define ranked models: primary → secondary → emergency fallback. Example: `claude-sonnet-4 → claude-haiku-3 → gpt-4o-mini`.
- **Trigger conditions for fallback**: HTTP 429 (rate limit), HTTP 503 (service unavailable), HTTP 500 (provider error), timeout exceeded, or structured output parse failure.
- **Retry with exponential backoff before fallback**: Distinguish transient errors (retry same model) from persistent failures (escalate). Minimum 3 attempts before switching models.
- **Output normalization after fallback**: Ensure outputs from all fallback models are normalized to the same schema before passing to downstream systems.
- **Log every fallback event**: Record the original model, reason for fallback, fallback model used, and outcome.
- **Circuit breaker pattern**: After N consecutive failures on a primary model within a rolling window, open the circuit breaker and route directly to the fallback for a configurable cooldown period.

---

## 6. Context Window Management Across Models

- **Always measure before sending**: Count tokens before constructing the final prompt. Use the target model's tokenizer. Never estimate by character count.
- **Define a context budget**: Allocate the window into segments: system prompt, retrieved context, conversation history, task instructions, output buffer.
- **History truncation strategy**: Truncate oldest turns first. Preserve the system prompt and most recent exchanges. Optionally replace truncated history with a rolling summary.
- **RAG for large codebases**: Do not stuff entire files into context. Retrieve only the most relevant code chunks via embedding search. Limit retrieved context to 30-40% of total window budget.
- **Cross-model context hand-off**: When moving from one model to another, serialize relevant context into a structured summary rather than passing raw history.
- **Monitor in production**: Log `prompt_tokens` and `completion_tokens` from every response. Alert when prompt tokens exceed 80% of the model's limit.

---

## 7. On-Device Models (Apple FoundationModels / iOS 26+)

For privacy-sensitive apps that must work offline, use Apple's on-device FoundationModels framework.

**Availability check (always required):**
```swift
struct GenerativeView: View {
    private var model = SystemLanguageModel.default

    var body: some View {
        switch model.availability {
        case .available:
            ContentView()
        case .unavailable(.deviceNotEligible):
            Text("Device not eligible for Apple Intelligence")
        case .unavailable(.appleIntelligenceNotEnabled):
            Text("Please enable Apple Intelligence in Settings")
        case .unavailable(.modelNotReady):
            Text("Model is downloading or not ready")
        case .unavailable(let other):
            Text("Model unavailable: \(other)")
        }
    }
}
```

**Basic session:**
```swift
let session = LanguageModelSession(instructions: """
    You are a cooking assistant. Provide brief, practical suggestions.
    """)
let response = try await session.respond(to: "I have chicken and rice")
```

**Structured output with `@Generable`:**
```swift
@Generable(description: "A trip idea")
struct TripIdea {
    var destination: String
    @Guide(description: "Why it's a good choice", .range(10...200))
    var rationale: String
}

let response = try await session.respond(to: "Suggest a trip", generating: TripIdea.self)
```

**Key constraints:**
- 4,096 token limit (instructions + prompt + output combined)
- One request per session at a time (`isResponding` check required)
- Access results via `.content`, not `.output`
- Use `@Generable` for structured output — stronger guarantees than parsing raw strings
- Snapshot streaming (not deltas) for real-time UI via `session.streamResponse(...)`

---

## 8. Consistent Output Format Regardless of Model

- **Schema-first design**: Define the output schema before selecting a model. Every model must produce output conforming to this schema.
- **Structured output enforcement**: Use JSON mode, structured output APIs, or tool-calling response format where available.
- **Output parsing and validation layer**: Implement a validation layer between the model response and your application that parses, validates against schema, and raises errors on malformed output.
- **Normalization of model-specific quirks**: Some models prepend preamble text or wrap JSON in markdown fences. Implement model-specific output normalization handlers.
- **Fallback parsing**: If strict parsing fails, use a lenient fallback parser. Log all fallback parse events.
- **Version your output schemas**: When changing the expected output format, version the schema and maintain backward-compatible parsing.

---

## 9. Model Versioning and Upgrades

- **Pin model versions in production**: Never use floating aliases (e.g., `gpt-4o-latest`, `claude-3-sonnet`). Always pin to a specific dated version (e.g., `claude-sonnet-4-20250514`).
- **Upgrade tracking**: Maintain a model version registry mapping task types to pinned versions. Run your test suite before deploying upgrades.
- **Canary deployments for model upgrades**: Roll out model version changes to 1-5% of traffic first. Monitor output quality metrics before increasing rollout.
- **Regression testing on upgrade**: Run your full prompt test suite against the new version before promoting it.
- **Deprecation monitoring**: Subscribe to provider deprecation notices. Alert when a pinned version approaches its deprecation date minus 30 days.
- **Rollback capability**: Maintain the ability to roll back to the previous model version within 5 minutes of detecting a production regression.

---

## 10. Latency Considerations

- **First-token latency vs. total latency**: For streaming user-facing responses, TTFT is the metric that drives perceived responsiveness. For batch pipelines, total latency matters more.
- **Parallel model calls**: When a task can be decomposed into independent subtasks, run them in parallel across models simultaneously.
- **Streaming for user-facing paths**: Always stream responses in user-facing applications. Implement streaming-compatible output parsers.
- **Pre-computation and prompt caching**: Use provider-side prompt caching (Anthropic prompt caching, OpenAI cached prompts) for repeated system prompt prefixes. Reduces both cost and latency.
- **Timeout budgets**: Typical production timeouts: 5s for TTFT, 30s for total completion on frontier models. Fail fast and trigger fallback rather than waiting indefinitely.
- **Latency SLA monitoring**: Track p50, p95, and p99 latency per model per task type. Alert on p95 degradation.

---

## 11. API Key and Credential Management

- **Never hardcode API keys**: Keys must never appear in source code, configuration files, or log output.
- **Centralized secrets management**: Store all API keys in a dedicated secrets manager (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault, GCP Secret Manager).
- **Per-environment key isolation**: Maintain separate keys for development, staging, and production.
- **Provider key abstraction**: Abstract credentials behind a unified resolver: `get_credential("anthropic")` — not direct environment variable references. Enables central rotation without code changes.
- **Key inventory tracking**: Maintain an inventory of all API keys: provider, environment, creation date, rotation date, owning team. Review and prune unused keys quarterly.

---

## 12. Monitoring and Observability

- **Structured request/response logging**: Log every model call with: timestamp, model ID, task type, prompt token count, completion token count, latency, status code. Do not log raw prompt content without PII scrubbing.
- **Per-model metrics dashboards**: Track request volume, error rate, latency distribution, token usage, and cost per model.
- **Unified tracing**: Use OpenTelemetry distributed tracing spanning multiple model calls within a single user request.
- **Output quality metrics**: Track structured output parse success rate, schema validation pass rate, and fallback parse trigger rate.
- **Alerting thresholds**: Error rate > 1% (critical), p95 latency > 2× baseline (warning), cost > 110% of daily budget (warning), fallback trigger rate > 5% (warning).

---

## Quick Reference: Decision Checklist

When building or reviewing a multi-model system:

- [ ] Model selection is documented per task type with rationale
- [ ] Prompts are tested against all models in the portfolio
- [ ] Model versions are pinned (no floating aliases in production)
- [ ] Fallback chains are defined and tested for all critical paths
- [ ] Context window budgets are calculated and enforced
- [ ] Output schemas are versioned and validated at the parsing layer
- [ ] All API keys are stored in a secrets manager, not in code
- [ ] Latency SLAs are defined with p95/p99 alerting
- [ ] Cost caps are enforced at the routing layer
- [ ] Distributed tracing spans all model calls in a request chain
- [ ] Regression test suite runs in CI on every prompt or model change
