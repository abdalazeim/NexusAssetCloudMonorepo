┌──────────────────────────────────────────────────────────────┐
│  File        : SECURITY.md                                   │
│  Project     : NexusAsset Cloud                              │
│  Module      : Security                                      │
│  Version     : 1.0.0                                         │
│  Status      : Stable                                        │
│  Owner       : Security Engineering                          │
│  Last Update : 2026-11-08                                    │
│  Language    : English                                       │
└──────────────────────────────────────────────────────────────┘

╔════════════════════════════════════════════════════════════════╗
║  SECURITY POLICY                                               ║
║  Vulnerability Reporting and Security Expectations             ║
╚════════════════════════════════════════════════════════════════╝

## Table of Contents
1. [Reporting a Vulnerability](#1-reporting-a-vulnerability)
2. [Scope](#2-scope)
3. [Handling Process](#3-handling-process)
4. [Security Requirements](#4-security-requirements)
5. [Related Files](#5-related-files)

┌──────────────────────────────────────────────────────────┐
│  1. Reporting a Vulnerability                            │
└──────────────────────────────────────────────────────────┘
Report suspected vulnerabilities privately to `security@nexusasset.cloud`. Do not open public issues for sensitive findings.

Include: affected component, impact, reproduction steps, logs/screenshots (sanitized), and suggested remediation if available.

┌──────────────────────────────────────────────────────────┐
│  2. Scope                                                │
└──────────────────────────────────────────────────────────┘
This policy covers NexusAsset Cloud code, APIs, infrastructure configuration, CI/CD pipelines, container images, and documentation that may expose security-sensitive information.

┌──────────────────────────────────────────────────────────┐
│  3. Handling Process                                     │
└──────────────────────────────────────────────────────────┘
1. Acknowledge receipt.
2. Triage severity and impact.
3. Reproduce and contain.
4. Remediate and verify.
5. Document lessons learned and preventive controls.

╭─ WARNING ───────────────────────────────────────────────────╮
│ Never include production secrets, tokens, private keys, or  │
│ cross-tenant data in reports or test artifacts.             │
╰─────────────────────────────────────────────────────────────╯

┌──────────────────────────────────────────────────────────┐
│  4. Security Requirements                                │
└──────────────────────────────────────────────────────────┘
- Enforce tenant isolation (`tenant_id` + PostgreSQL RLS).
- Use least privilege and strong authentication/authorization.
- Store secrets in approved secret stores; use `NXA_*` names.
- Run dependency, secret, SAST, and DAST/security checks in CI/CD.
- Maintain audit logs for sensitive actions.

┌──────────────────────────────────────────────────────────┐
│  5. Related Files                                        │
└──────────────────────────────────────────────────────────┘
- [07-security/01-threat-model.md](./07-security/01-threat-model.md)
- [07-security/03-secrets-management.md](./07-security/03-secrets-management.md)
- [07-security/02-owasp-checklist.md](./07-security/02-owasp-checklist.md)

─────────────────────────────────────────────────────────────
End of file — NexusAsset Cloud
For questions: security@nexusasset.cloud
─────────────────────────────────────────────────────────────
