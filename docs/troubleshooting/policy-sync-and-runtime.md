# Policy Sync and Runtime Issues

Synchronization failures and runtime execution failures can look similar at the symptom layer, but they represent different operational causes.

## Common Symptoms

- new policy is not reflected
- one engine behaves differently from others
- Wrapper uses outdated mapping information
- encryption succeeds on one path but fails on another

## Diagnostic Questions

1. Was the policy source of truth actually updated in Hub?
2. Is the Engine runtime cache current?
3. Is the Wrapper local metadata current?
4. Which path fails: Wrapper, Direct API, or DB UDF?

## Operational Interpretation

- Hub to Engine sync failure means runtime cache is stale.
- Wrapper sync failure means local execution metadata is stale.
- Engine runtime failure means the execution path itself is impaired.
