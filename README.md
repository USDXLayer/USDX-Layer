![USDX Layer](usdxlayer.png)

# USDX Layer

USDX Layer is a deterministic execution and routing abstraction derived from USD1 usage.

USDX Layer defines how USD1-based economic activity is observed, normalized, routed, executed, and recorded without custody, wrapping, or modification of USD1 itself.

USD1 remains the stable unit of account.
USDX Layer defines the execution surface around it.

---

## Abstract

USDX Layer is a modular specification and reference implementation for deterministic execution routing of USD1 activity.

The system operates as a second-order layer that derives its inputs exclusively from observed USD1 economic events. It produces execution plans and accounting artifacts that are reproducible, auditable, and invariant under identical inputs.

USDX Layer is designed to be non-custodial, chain-agnostic, and suitable for use as an execution SDK, reference architecture, or research substrate for higher-order USD-derived systems.

The core objective of USDX Layer is to reduce ambiguity in execution behavior while preserving composability and predictability.

---

## Motivation

USD1 establishes a stable monetary primitive.

As adoption increases, higher-order systems emerge that depend not on the value of USD1 itself, but on how USD1 moves through execution paths.

Without a formal execution abstraction, routing behavior becomes fragmented, opaque, and difficult to reason about across systems.

USDX Layer exists to formalize execution behavior without interfering with monetary stability.

It answers the question:

How should USD1 activity move through systems in a way that is deterministic, observable, and verifiable?

---

## Scope and Non Goals

### In Scope

- Observation and normalization of USD1 activity
- Deterministic routing over constrained execution graphs
- Execution plan generation
- Settlement simulation
- Hash chained accounting journals
- Invariant enforcement and verification
- Simulation and testing of execution behavior

### Explicit Non Goals

- USDX Layer is not a bridge
- USDX Layer is not a wrapper for USD1
- USDX Layer does not custody funds
- USDX Layer does not introduce yield mechanics
- USDX Layer does not define governance over USD1
- USDX Layer does not assume any specific blockchain deployment

---

## Design Principles

USDX Layer is guided by the following principles.

### Determinism

Identical inputs must produce identical outputs.

This includes:
- Event ordering
- Path enumeration
- Path scoring
- Execution planning
- Settlement artifacts
- Journal hashes

### Separation of Concerns

Each stage of the system is isolated.

Observation does not route.
Routing does not execute.
Execution does not account.
Accounting does not decide.

### Non Custodial Design

No component requires holding, escrowing, or modifying USD1 balances.

### Auditability

All execution artifacts are recorded in an append-only journal with cryptographic integrity.

### Minimal Assumptions

The system avoids assumptions about chain state, settlement finality, or external pricing.

---

## System Architecture

USDX Layer is composed of four primary layers.

(Architecture diagram omitted here for brevity; see full repo for details.)

---

## License

MIT License.
