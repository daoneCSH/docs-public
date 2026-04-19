# License and Trust Issues

License and trust problems should be treated as operating-state failures rather than as generic API errors. This category includes validation, bootstrap recovery, certificate alignment, and service trust failures.

## Common Symptoms

- license validation fails
- validation succeeds but status remains disconnected
- mTLS channel fails after certificate change
- connected operation falls back to bootstrap recovery

## First Checks

1. connected-mode validation response
2. license or trust status channel state
3. bootstrap path availability
4. certificate and truststore alignment
5. connected or air-gapped operating mode

## Operational Interpretation

- Validation success does not guarantee a stable connected state.
- mTLS failure should be interpreted as a service trust problem.
- Air-gapped recovery follows a different operational rule set from connected recovery.
