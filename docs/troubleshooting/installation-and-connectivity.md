# Installation and Connectivity Issues

Installation and connectivity issues indicate that a component has not yet reached a stable operating boundary.

## Common Symptoms

- service process does not start
- required port does not respond
- Wrapper cannot reach Hub
- DB UDF cannot reach Engine
- reverse proxy or load balancer does not route correctly

## First Checks

1. process or container state
2. environment variables and secrets
3. target URL and port
4. network and firewall rules
5. database-side ACL or outbound connectivity

## Typical Interpretation

- Hub startup failures usually indicate control-plane configuration errors.
- Engine reachability failures usually indicate URL, port, or TLS problems.
- DB UDF path failures should be checked against engine endpoint, ACL, and database object state.
