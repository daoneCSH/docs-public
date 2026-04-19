# Configuration Overview

DADP configuration should be managed by module responsibility rather than as one flat list of environment variables.

## Configuration Domains

| Domain | Primary Owner | Description |
|--------|---------------|-------------|
| control-plane configuration | Hub | policy, operations, metadata, and connected-mode settings |
| data-plane configuration | Engine | runtime ports, providers, cache, and execution behavior |
| integration configuration | Wrapper, DB UDF | Hub URL, Engine URL, and path-specific settings |
| infrastructure configuration | Proxy, Load Balancer, DNS | operator access TLS, routing, and network boundaries |

## Configuration Rules

1. End-user HTTPS and service trust configuration should be managed separately.
2. Hub owns control-plane settings.
3. Engine owns runtime settings.
4. Wrapper and DB UDF should keep only the execution settings they require.
