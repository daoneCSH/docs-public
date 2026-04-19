# Connected and Air-Gapped Modes

DADP deployment mode is defined by operating connectivity, not by infrastructure label alone. Public documentation uses the terms `Connected` and `Air-Gapped` because trust, validation, and recovery behave differently in each mode.

## Connected Mode

In connected mode, Hub operates with upstream-connected control dependencies and trust-distribution paths.

### Characteristics

- connected control services are reachable
- validation and trust state are interpreted through connected operating paths
- bootstrap and steady-state channels are separated

## Air-Gapped Mode

In air-gapped mode, Hub operates without continuous upstream connectivity.

### Characteristics

- trust assets and validation material are handled inside the deployment boundary
- local operating state is interpreted within the deployment itself
- recovery follows deployment-local procedures

## Operational Interpretation

The difference between connected and air-gapped mode is not just location. It changes how trust, validation, and recovery state are established and interpreted.
