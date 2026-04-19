# Platform Overview

DADP is a data-protection platform that separates policy control from runtime execution. Hub manages policy and operational state, while Engine processes cryptographic requests. Wrapper, Direct API, and DB UDF connect application and database execution paths to the runtime layer.

## Platform Layers

| Layer | Primary Components | Role |
|------|--------------------|------|
| Control Plane | Hub | policy, key metadata, integration mapping, and operations |
| Data Plane | Engine, Aggregator | cryptographic execution and optional analytics |
| Integration Layer | Wrapper, Direct API, DB UDF | path-specific integration into applications and databases |

## Operating Principles

1. Policy is authored in the control plane.
2. Cryptographic execution is handled in the data plane.
3. Integration methods change the entry point, not the execution owner.

## Official Integration Paths

| Method | Entry Point | Best Fit |
|--------|-------------|----------|
| Wrapper | JDBC boundary | application-side database access protection |
| Direct API | application service code | explicit runtime invocation from code |
| DB UDF | database SQL path | database-side execution through SQL |

## Next Reading

- [Core Concepts](core-concepts.md)
- [Platform Architecture](../architecture/platform-architecture.md)
- [Integration Overview](../integrations/integration-overview.md)
