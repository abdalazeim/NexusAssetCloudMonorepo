# NexusAsset Cloud — Rate Limiting and Throttling

## Status
Target/Unverified.

## Requirements (Target)
- Apply rate limits at edge (`nxa-traefik-public`) and/or application as approved.
- Protect login, webhook, sync-trigger, and expensive list/search endpoints.
- Return consistent `429` responses with safe retry guidance.
- Use tenant/user/IP/device dimensions as appropriate; do not create cross-tenant side channels.
- Monitor throttling metrics and abuse patterns.

## Evidence Needed
Traefik/app config, load/security tests, and operational dashboards/alerts.
