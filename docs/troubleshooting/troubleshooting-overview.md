# Troubleshooting Overview

DADP troubleshooting is organized by operational failure boundary. Installation, connectivity, synchronization, runtime execution, and trust issues should not be handled as one undifferentiated problem class.

## Problem Categories

| Category | Typical Scope |
|----------|---------------|
| Installation | startup failure, missing configuration, object creation failure |
| Connectivity | network path, URL, TLS, ACL, DNS |
| Synchronization | policy distribution and runtime state mismatch |
| Runtime | encryption or decryption execution failure |
| License and Trust | bootstrap, validation, certificate, and mTLS issues |

## Diagnostic Rule

When triaging an issue, identify which boundary fails first:

1. control plane
2. data plane
3. integration path
4. trust and licensing
