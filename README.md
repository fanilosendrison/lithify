---
okf_version: "1.0"
kind: "KnowledgeAsset"
asset_type: "documentation"
domain: "lithify"
severity: "guideline"
name: "Lithify"
version: "0.1.0"
---

# Lithify

Lithify provides reliable, backend-independent mechanical primitives for durable engineering work state and records. Autonomous callers use Lithify to establish explicitly requested work state, publish and retrieve engineering work records, and verify the resulting backend facts. These capabilities allow agent instructions, execution reports, and related work information to remain accessible across executions and context boundaries without relying on transient agent memory.

Lithify does not decide what work should exist, what information should be published, how reports should be interpreted, or what execution should happen next. GitHub is the initial backend, not the product definition. There is no Lithify implementation yet.

## Normative specification

The normative product authority is
[the Lithify specification](docs/specification/lithify-spec.md). Its Section 0
states the product intent that every later requirement, architectural
implication, and implementation must preserve.

## Development method and Proto-Lithify

Lithify's necessary product obligations are derived independently from its
accepted Product Intent. Proto-Lithify is a separate empirical GitHub prototype,
not a required architecture or a normative reference implementation.

The [specification discipline](docs/specification/lithify-spec.md#1-specification-discipline)
defines independent derivation, explicit product choices, and the later bounded
confrontation of established obligations with empirical evidence.
