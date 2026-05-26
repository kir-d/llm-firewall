---
description: Understand workspaces, projects, and how permissions work across the platform.
icon: spider-black-widow
---

# Blocking Threats

CollieAI provides five rule types for detecting and blocking threats in LLM traffic. Four operate on inputs (prompt injection, malicious URLs, encoded payloads); one — Streaming Guard — operates on outputs as the model response streams.

### [Rule Types](https://app.collieai.io/app/docs/security-rules/blocking-threats#rule-types) <a href="#rule-types" id="rule-types"></a>

#### [Prompt Injection Detection](https://app.collieai.io/app/docs/security-rules/prompt-injection)

Lightweight ML classifier that scores messages for prompt injection and jailbreak attempts. Fast inference (10-50ms) with multiple pre-trained models available.

**Use for:** High-throughput injection detection, first line of defense.

#### [LLM Detection](https://app.collieai.io/app/docs/security-rules/llm-detection)

Generative LLM-based detection that provides structured JSON output with reasoning. Better at catching novel attacks and explaining why a message was flagged.

**Use for:** High-accuracy detection, novel attack patterns, audit trails requiring explainability.

#### [URL Filtering](url-filtering.md)

Extracts, normalizes, and filters URLs found in messages. Supports domain allowlists/blocklists, scheme restrictions, IP literal blocking, and encoded pattern detection.

**Use for:** Preventing exfiltration via URLs, blocking phishing links, enforcing trusted domains.

#### [Base64 Payload Detection](base64-payloads.md)

Detects base64-encoded content including data URIs and raw base64 strings. Identifies file types by signature and enforces size limits.

**Use for:** Blocking encoded executables, enforcing payload size limits, controlling file types in messages.

#### [Streaming Guard](streaming-guard.md)

Output-only safety guard backed by Qwen3Guard-Stream. Classifies model output chunks across 8 risk categories (Violent, Sexual Content, Self-Harm, PII, etc.) and enforces per-category block / mask / monitor decisions in real time as the response streams.

**Use for:** Customer-facing chatbots where unsafe output is a brand risk, streaming endpoints where waiting for the full response before enforcing isn't acceptable, policies needing fine-grained per-category control on the output side.

### Recommended Rule Order <a href="#recommended-rule-order" id="recommended-rule-order"></a>

When combining threat detection rules, place **ML model rules before normalization** in your rule order. ML models are trained on natural text and perform best on the original, un-normalized input.

A typical ordering:

| Order | Rule                                  | Why                             |
| ----- | ------------------------------------- | ------------------------------- |
| 1-5   | Prompt Injection (lightweight\_model) | Evaluate original text          |
| 5-10  | LLM Detection                         | Evaluate original text          |
| 10-15 | Normalization                         | Clean text for downstream rules |
| 15-20 | URL Filtering                         | Works on normalized text        |
| 20-25 | Base64 Payload Detection              | Works on normalized text        |
| 25+   | PII Detection rules                   | Works on normalized text        |

Adjust ordering based on your specific requirements. The key principle is that ML-based classifiers should see the message before text normalization transforms it.
