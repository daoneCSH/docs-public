# Security Overview

DADP security is organized into distinct layers instead of a single blended model. User authentication, authorization, and service trust are documented separately because they operate at different boundaries.

## Security Layers

| Layer | Primary Subject | Purpose |
|-------|-----------------|---------|
| User Authentication | browser user or API user | verify identity and establish sessions |
| Authorization | Hub role model | enforce feature access |
| Service Trust | service instances | establish service-to-service trust |

## Core Principles

1. User authentication and service authentication are separate concerns.
2. Token validity and feature authorization are evaluated independently.
3. Bootstrap recovery paths and normal operating paths are documented separately.

## Reading Guide

- [Authentication and Access Control](authentication-and-access-control.md)
- [Trust and Certificates](trust-and-certificates.md)
- [Connected and Air-Gapped Authentication](connected-and-air-gapped-authentication.md)
