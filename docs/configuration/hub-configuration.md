# Hub Configuration

Hub configuration exists to keep the DADP control plane stable and predictable in operation.

## Primary Configuration Areas

- server port and context path
- Meta DB connectivity
- Redis connectivity
- connected-mode upstream endpoints, when applicable
- log level
- integration management settings
- analytics collection settings

## Operational Interpretation

- Hub is the source of truth for policy and operational state, so metadata and dependency connectivity must remain stable.
- Integration instances should be managed through operational paths rather than by ad hoc runtime mutation.
- CORS, TLS, and same-origin deployment rules should be evaluated together at the operator entry boundary.
