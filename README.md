# USD1 Routing

![USDX Layer](./usd1routing.png)

## Abstract

USD1 Routing is an execution and routing abstraction layer designed to derive deterministic execution paths from observed USD1 economic activity. It does not custody, wrap, or modify USD1 tokens. Instead, it provides a formal framework for defining and enforcing routing policies, execution constraints, and integrity verification for USD1-based operations.

The system operates as a pure abstraction: it consumes normalized observations of USD1 activity, applies deterministic routing logic through a constraint graph, and produces verifiable execution plans. All execution paths are reproducible from the same input state and policy configuration.

## Motivation

USD1 activity occurs across multiple protocols, chains, and execution environments. Without a shared abstraction layer, routing decisions are ad-hoc, non-reproducible, and difficult to audit. USDX Layer addresses this by:

1. Normalizing observed USD1 activity into a canonical event stream.
2. Defining a deterministic routing graph with explicit policy constraints.
3. Computing execution paths that satisfy invariants and minimize unnecessary state transitions.
4. Providing cryptographic integrity proofs for all routing decisions.

### Non-Goals

This layer does not:

- Custody or hold USD1 tokens
- Wrap or modify USD1 in any way
- Guarantee yield or returns
- Execute transactions on-chain directly
- Provide price oracles or market data

## Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                     Observation Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Volume     │  │    Events    │  │  Normalizer  │      │
│  │   Monitor    │  │   Collector  │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────────────┬────────────────────────────────────┘
                         │ Normalized Activity Stream
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                      Routing Layer                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │    Graph     │  │    Policy    │  │    Paths     │      │
│  │  Structure   │  │  Constraints │  │   Resolver   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                   ┌──────────────┐                          │
│                   │ Constraints  │                          │
│                   │   Validator  │                          │
│                   └──────────────┘                          │
└────────────────────────┬────────────────────────────────────┘
                         │ Execution Plan
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                     Execution Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │    Engine    │  │    Hooks     │  │   Settle     │      │
│  │              │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                   ┌──────────────┐                          │
│                   │ Deterministic│                          │
│                   │   Verifier   │                          │
│                   └──────────────┘                          │
└────────────────────────┬────────────────────────────────────┘
                         │ Ledger Entries
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    Accounting Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │    Ledger    │  │   Journal    │  │  Integrity   │      │
│  │              │  │              │  │   Checker    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

## Module Overview

### Observation

The observation layer normalizes heterogeneous USD1 activity into a canonical event stream. It does not interact with chains directly but consumes pre-collected data.

- **volume.ts**: Aggregates volume metrics per time window
- **events.ts**: Defines event types and schemas
- **normalizer.ts**: Transforms raw activity into normalized events

### Routing

The routing layer maintains a directed graph of valid execution paths and selects paths based on policy constraints.

- **graph.ts**: Graph data structure and traversal
- **policy.ts**: Policy definition and evaluation
- **paths.ts**: Path selection algorithm
- **constraints.ts**: Constraint types and validators

### Execution

The execution layer consumes routing decisions and produces deterministic execution plans. Plans are represented as ordered sequences of primitive operations.

- **engine.ts**: Main execution engine
- **hooks.ts**: Pre and post execution hooks
- **deterministic.ts**: Determinism verification
- **settle.ts**: Settlement logic

### Accounting

The accounting layer records all execution steps in an append-only ledger and provides integrity verification.

- **ledger.ts**: Ledger structure and append operations
- **journal.ts**: Journal entries and queries
- **integrity.ts**: Hash chain and integrity proofs

### Simulation

The simulation layer generates synthetic USD1 activity and measures system behavior under various scenarios.

- **montecarlo.ts**: Monte Carlo simulation engine
- **scenarios.ts**: Predefined test scenarios
- **runner.ts**: Simulation orchestration

### Utils

- **hash.ts**: Cryptographic hashing utilities
- **math.ts**: Fixed-point arithmetic and statistical functions
- **time.ts**: Timestamp normalization
- **validate.ts**: Input validation

## Deterministic Routing Model

Routing decisions are deterministic functions of:

1. Current graph state G
2. Observed activity event E
3. Policy configuration P
4. Historical context H (last N events)

Given the same inputs, the routing engine always produces the same output. This property is verified through hash-based integrity checks.

### Path Selection Algorithm
```
function selectPath(G, E, P, H):
  candidates = G.outgoingEdges(E.source)
  
  for edge in candidates:
    if not satisfiesConstraints(edge, E, P, H):
      candidates.remove(edge)
  
  if candidates.isEmpty():
    return null
  
  return deterministicSelect(candidates, hash(E, H))
```

The deterministic selection function uses a hash of the event and context as a seed for consistent tie-breaking.

## Execution Lifecycle

1. **Observation**: Activity event enters the system
2. **Normalization**: Event is transformed into canonical format
3. **Routing**: Graph traversal produces candidate paths
4. **Constraint Validation**: Paths are filtered by policy
5. **Path Selection**: Deterministic selection from valid candidates
6. **Execution Planning**: Selected path is expanded into primitive operations
7. **Integrity Verification**: Plan hash is computed and stored
8. **Settlement**: Plan is marked as settled in the ledger

## Constraints and Invariants

### Core Invariants

1. **Determinism**: Identical inputs produce identical outputs
2. **Conservation**: No value is created or destroyed within the layer
3. **Acyclicity**: Execution plans contain no cycles
4. **Monotonicity**: Ledger is append-only
5. **Integrity**: All entries are hash-linked

See [FORMAL_INVARIANTS.md](./FORMAL_INVARIANTS.md) for detailed specifications.

## Threat Model Summary

The system is designed to resist:

- **State manipulation**: Ledger integrity prevents historical rewrites
- **Non-determinism injection**: Execution engine enforces reproducibility
- **Policy bypass**: Constraint validators are mandatory
- **Volume inflation**: Observation layer normalizes inputs

The system does NOT protect against:

- Malicious input data (garbage in, garbage out)
- External protocol failures
- Key compromise of upstream systems

See [THREAT_MODEL.md](./THREAT_MODEL.md) for comprehensive analysis.

## Usage Example
```typescript
import { USDXLayer } from 'usdx-layer';
import { ActivityEvent } from 'usdx-layer/types';

// Initialize the layer with a policy configuration
const layer = new USDXLayer({
  policy: {
    maxPathLength: 5,
    allowedProtocols: ['protocol-a', 'protocol-b'],
    minVolumeThreshold: 1000000n, // 1M units
  },
});

// Submit an observed activity event
const event: ActivityEvent = {
  id: 'evt_001',
  timestamp: 1704067200000,
  source: 'protocol-a',
  destination: 'protocol-b',
  volume: 5000000n,
  metadata: {},
};

// Compute execution plan
const result = await layer.route(event);

if (result.success) {
  console.log('Execution plan:', result.plan);
  console.log('Integrity proof:', result.proof);
  
  // Verify determinism
  const result2 = await layer.route(event);
  assert(result.proof.hash === result2.proof.hash);
} else {
  console.error('Routing failed:', result.error);
}
```

## Simulation

The simulation module can generate synthetic USD1 activity and measure routing performance.
```typescript
import { runSimulation } from 'usdx-layer/simulation';

const results = await runSimulation({
  scenario: 'high-volume',
  duration: 86400000, // 24 hours
  eventRate: 100, // events per second
  seed: 12345,
});

console.log('Total events:', results.eventCount);
console.log('Successful routes:', results.successCount);
console.log('Failed routes:', results.failureCount);
console.log('Average path length:', results.avgPathLength);
console.log('Determinism violations:', results.determinismViolations);
```

### Example Simulation Output
```
Scenario: high-volume
Duration: 86400000ms (24h)
Event rate: 100/s
Seed: 12345

Results:
  Total events: 8640000
  Successful routes: 8639234 (99.99%)
  Failed routes: 766 (0.01%)
  Average path length: 2.3
  Median path length: 2
  Max path length: 5
  Determinism violations: 0
  
Constraint violations by type:
  maxPathLength: 512
  volumeThreshold: 254
  
Execution time: 324.5s
Events per second: 26620
```

## Testing Strategy

The test suite validates:

1. **Determinism**: Identical inputs produce identical outputs across multiple runs
2. **Invariants**: All formal invariants hold under normal and adversarial inputs
3. **Constraints**: Policy constraints are correctly enforced
4. **Integrity**: Ledger hash chains are unbroken
5. **Simulation**: Monte Carlo tests cover edge cases

Tests are located in the `test/` directory and can be run with:
```bash
npm test
```

## Running Locally

### Prerequisites

- Node.js 20 or higher
- npm 10 or higher

### Installation
```bash
git clone https://github.com/your-org/usdx-layer.git
cd usdx-layer
npm install
```

### Build
```bash
npm run build
```

### Run Tests
```bash
npm test
```

### Run Simulation
```bash
npm run sim
```

### Type Check
```bash
npm run typecheck
```

### Lint
```bash
npm run lint
```

## Project Status

This is research and development software. It is not audited and should not be used in production.

## License

MIT License. See [LICENSE](./LICENSE) for details.

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

## Security

See [SECURITY.md](./SECURITY.md) for reporting vulnerabilities.
```

---

### LICENSE
```
MIT License

Copyright (c) 2024 USDX Layer Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
