# Core Concepts

This page introduces the core operating concepts used throughout DADP public documentation.

## Control Plane

The control plane manages policy, key metadata, integration mapping, and operational state. In public documentation, Hub is the central control-plane component.

## Data Plane

The data plane executes encryption, decryption, and masking requests. Engine is the runtime execution component of the data plane.

## Runtime Cache

Engine does not author policy. It executes with the runtime state synchronized from Hub and stored as execution-ready cache.

## Integration Layer

The integration layer connects application or database execution paths to Engine. DADP provides three primary integration paths:

- Wrapper
- Direct API
- DB UDF

## Source of Truth

| Information | Source of Truth | Runtime Consumption |
|-------------|-----------------|---------------------|
| policy and key metadata | Hub | Engine runtime cache |
| integration mapping | Hub | Wrapper or integration helper state |
| operational mode | deployment configuration | Hub and integration behavior |

## Operating Rules

- Do not treat control-plane issues and data-plane issues as the same class of problem.
- Do not confuse runtime failure with policy propagation delay.
- Distinguish integration-path differences from execution-engine behavior.

## Next Reading

- [Platform Overview](platform-overview.md)
- [Platform Architecture](../architecture/platform-architecture.md)
- [Module Overview](../modules/module-overview.md)
