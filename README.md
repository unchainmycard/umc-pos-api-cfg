# umc-pos-api-cfg

Configuration service for the UMC POS solution.

## Overview

This module manages configuration data for multi-tenant deployments and environment-specific settings. It provides a centralized configuration layer that enables flexible deployment across different retail chains and store configurations.

## Key Responsibilities

- **Tenant Configuration**: Chain-specific settings, branding, business rules
- **Store Profiles**: Per-store configurations, register assignments
- **Feature Flags**: Enable/disable features per tenant or store
- **Environment Settings**: Dev, staging, production configurations
- **Secrets Management**: Secure storage for API keys and credentials

## Configuration Scope

| Level | Examples |
|-------|----------|
| Global | System defaults, shared resources |
| Tenant | Chain branding, business rules, integrations |
| Store | Local settings, hardware profiles |
| Register | Device-specific configurations |

## Design Principles

- **Hierarchical**: Configurations cascade from global to specific
- **Versioned**: Track configuration changes over time
- **Cacheable**: Optimized for fast reads by client applications

## Part of UMC POS

This module is part of the [umc-pos-workspace](https://github.com/your-org/umc-pos-workspace) - a unified modern POS solution for retail chains in France and internationally.

| Related Module | Purpose |
|----------------|---------|
| `umc-pos-web-admin` | Back-office administration |
| `umc-pos-web-vend` | Point of sale interface |
| `umc-pos-api-svc` | Backend API services |
