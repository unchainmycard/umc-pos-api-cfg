# UMC POS Configuration Repository

Centralized configuration for all UMC POS microservices, served by Spring Cloud Config Server.

**Reference**: [ADR-P-config-service-020](../docs/adr/product/ADR-P-config-service-020.md)

---

## Repository Structure

```
umc-pos-api-cfg/
├── README.md                     # This file
├── application.yml               # Global defaults (all services)
├── {service}.yml                 # Service-specific defaults
│
└── levels/                       # Hierarchical configuration
    ├── tenants/                  # Level 1: Tenant configs
    │   └── application-{tenant}.yml
    │
    ├── regions/                  # Level 2: Region configs (optional)
    │   └── {tenant}/
    │       └── application-{tenant}-region-{region}.yml
    │
    ├── stores/                   # Level 3: Store configs
    │   └── {tenant}/
    │       └── application-{tenant}-store-{store}.yml
    │
    ├── zones/                    # Level 4: Zone configs (optional)
    │   └── {tenant}/{store}/
    │       └── application-{tenant}-store-{store}-zone-{zone}.yml
    │
    └── stations/                 # Level 5: Station configs
        └── {tenant}/{store}/{zone}/
            └── application-{tenant}-store-{store}-zone-{zone}-{station}.yml
```

---

## Profile Chain Mechanism

Services load configuration based on their `CONFIG_PROFILES` environment variable. Profiles are comma-separated and resolved in order, with later profiles overriding earlier ones.

### Example Profile Chain

```bash
CONFIG_PROFILES=demo-tenant,store-001,zone-checkout,reg-001
```

**Resolution order** (each level overrides previous):

| Order | Profile | File |
|-------|---------|------|
| 1 | (base) | `application.yml` |
| 2 | (base) | `{service}.yml` |
| 3 | demo-tenant | `levels/tenants/application-demo-tenant.yml` |
| 4 | store-001 | `levels/stores/demo-tenant/application-demo-tenant-store-001.yml` |
| 5 | zone-checkout | `levels/zones/.../application-demo-tenant-store-001-zone-checkout.yml` |
| 6 | reg-001 | `levels/stations/.../application-demo-tenant-store-001-zone-checkout-reg-001.yml` |

---

## Profile Naming Conventions

| Level | Profile Pattern | Example | Description |
|-------|-----------------|---------|-------------|
| Tenant | `{tenant-id}` | `carrefour`, `auchan` | Organizational identifier |
| Region | `region-{region-id}` | `region-idf`, `region-paca` | Geographic grouping |
| Store | `store-{store-id}` | `store-0042` | Physical store |
| Zone | `zone-{zone-id}` | `zone-checkout`, `zone-sco` | Functional area in store |
| Station | `{type}-{id}` | `reg-001`, `sco-003`, `tablet-007` | Individual device |

### Station Types

| Type | Description |
|------|-------------|
| `reg-XXX` | Traditional cashier register |
| `sco-XXX` | Self-checkout kiosk |
| `tablet-XXX` | Mobile POS tablet |
| `kiosk-XXX` | Information/price-check kiosk |

---

## Configuration Levels

### Level 1: Tenant

Business-wide configuration shared across all stores.

```yaml
# levels/tenants/application-{tenant}.yml
tenant:
  id: carrefour
  country: FR
  currency: EUR
  fiscal:
    regime: NF525
```

### Level 2: Region (Optional)

Regional overrides for geographic groupings.

```yaml
# levels/regions/{tenant}/application-{tenant}-region-{region}.yml
region:
  id: region-idf
  timezone: Europe/Paris
```

### Level 3: Store

Store-specific settings including local infrastructure.

```yaml
# levels/stores/{tenant}/application-{tenant}-store-{store}.yml
store:
  id: store-0042
  name: Paris Centre
  infrastructure:
    registers: 12
```

### Level 4: Zone (Optional)

Functional area configuration (checkout, self-checkout, backroom).

```yaml
# levels/zones/.../application-...-zone-{zone}.yml
zone:
  id: zone-checkout
  type: cashier
checkout:
  payment-methods:
    cash: true
    card: true
```

### Level 5: Station

Individual device configuration with specific peripherals.

```yaml
# levels/stations/.../application-...-{station}.yml
station:
  id: reg-001
server:
  port: 8081
peripherals:
  receipt-printer:
    host: 192.168.1.101
```

---

## Service Configuration

Each service can have its own default configuration:

| File | Description |
|------|-------------|
| `application.yml` | Global defaults for all services |
| `api-gateway-service.yml` | API Gateway specific config |
| `audit-service.yml` | Audit Service specific config |
| `discovery-service.yml` | Eureka Server specific config |

---

## Usage

### Docker Compose Example

```yaml
services:
  api-gateway-reg001:
    image: umc-pos/api-gateway:latest
    environment:
      SPRING_PROFILES_ACTIVE: docker
      CONFIG_PROFILES: demo-tenant,store-001,zone-checkout,reg-001
      CONFIG_SERVER_HOST: config-service
```

### Local Development

```bash
# Start with tenant + store profiles
export CONFIG_PROFILES=demo-tenant,store-001
./mvnw spring-boot:run
```

---

## Adding New Configuration

### New Tenant

1. Create `levels/tenants/application-{tenant}.yml`
2. Define tenant-wide settings (fiscal, loyalty, features)
3. Use profile: `CONFIG_PROFILES={tenant},...`

### New Store

1. Create `levels/stores/{tenant}/application-{tenant}-store-{store}.yml`
2. Define store-specific settings (hours, peripherals, infrastructure)
3. Use profile: `CONFIG_PROFILES={tenant},store-{store},...`

### New Station

1. Create `levels/stations/.../application-...-{station}.yml`
2. Define station-specific settings (port, peripheral IPs)
3. Use profile: `CONFIG_PROFILES={tenant},store-{store},zone-{zone},{station}`

---

## Troubleshooting

### Check Active Profiles

```bash
curl http://localhost:8080/actuator/env | jq '.activeProfiles'
```

### Verify Config Resolution

```bash
curl -u config:config http://localhost:8888/{service}/{profiles}/main
```

### Profile Not Applied?

1. Verify `CONFIG_PROFILES` environment variable is set
2. Check file naming matches profile exactly
3. Ensure file is committed to git (Config Server reads from git)
4. Restart Config Server after adding new files

---

## Related Documentation

- [ADR-P-config-service-020](../docs/adr/product/ADR-P-config-service-020.md) — Configuration strategy
- [ADR-P-deployment-016](../docs/adr/product/ADR-P-deployment-016.md) — Tier 1 edge deployment
- [ADR-P-service-discovery-021](../docs/adr/product/ADR-P-service-discovery-021.md) — Eureka zones

---

## Part of UMC POS

This module is part of the [umc-pos-workspace](https://github.com/unchainmycard/umc-pos-workspace) — a unified modern POS solution for retail chains in France and internationally.

| Related Module | Purpose |
|----------------|---------|
| `umc-pos-web-admin` | Back-office administration |
| `umc-pos-web-vend` | Point of sale interface |
| `umc-pos-api-svc` | Backend API services |
