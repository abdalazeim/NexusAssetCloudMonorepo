# DOC-16: Compliance & Audit Framework

---

| **Document ID** | DOC-16 |
|-----------------|--------|
| **Title** | Compliance & Audit Framework |
| **Version** | 1.0.0 |
| **Status** | ✓ Approved |
| **Owner** | Security & Compliance |
| **Last Updated** | 2026-11-09 |
| **Classification** | Internal — Confidential |
| **Related Docs** | [DOC-04 Domain Model](../00-overview/04-domain-model.md), [DOC-06 Infrastructure Architecture](../00-overview/06-infrastructure-architecture.md), [DOC-12 Security Architecture](../00-overview/12-security-architecture.md), [DOC-13 Observability](../00-overview/13-observability.md), [DOC-14 Data Migration](../00-overview/14-data-migration-seed-strategy.md), [DOC-15 Operations Runbook](../00-overview/15-operations-runbook.md) |
| **Traceability** | REQ-NFR-COMP-###, REQ-SEC-###, ADR-037..042 |

---

## 1. Executive Summary

This document defines the **compliance and audit framework** for NexusAsset Cloud, covering **SOC 2 Type II**, **ISO 27001**, **GDPR**, **HIPAA** (future), and **industry-specific** requirements. It establishes **evidence collection automation**, **continuous compliance monitoring**, **audit readiness procedures**, and **regulatory mapping** to technical controls. The framework ensures **audit-ready posture at all times** with **< 2 week audit preparation**.

> **Note:** Compliance is not a point-in-time activity. Continuous compliance monitoring via **automated evidence collection**, **drift detection**, and **policy-as-code** ensures perpetual readiness.

---

## 2. Regulatory Scope & Applicability

### 2.1 Compliance Framework Coverage

| Framework | Status | Scope | Audit Cycle | Evidence Owner |
|-----------|--------|-------|-------------|----------------|
| **SOC 2 Type II** | ✅ Active | Security, Availability, Confidentiality | Annual (12-month) | Security & Compliance |
| **ISO 27001:2022** | ✅ Active | ISMS (all domains) | Annual surveillance, 3-year recert | Security & Compliance |
| **GDPR** | ✅ Active | Personal data processing (EU/UK tenants) | Ongoing | DPO / Legal |
| **HIPAA** | 📋 Planned (Q2 2027) | PHI if healthcare tenants onboarded | Annual | Security & Compliance |
| **PCI DSS SAQ A** | 📋 Planned (Q3 2027) | If payment processing added | Annual | Security & Finance |
| **NIST 800-53 Rev 5** | ✅ Mapped | FedRAMP Medium baseline (future) | Continuous | Security & Compliance |

### 2.2 Control Mapping Matrix

```ascii
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    COMPLIANCE CONTROL MAPPING                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  NEXUSASSET CONTROLS                                                            │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  CC-01: Access Control (RBAC/ABAC)                                      │   │
│  │  CC-02: Encryption (At Rest / In Transit)                               │   │
│  │  CC-03: Audit Logging (Immutable, Tamper-evident)                       │   │
│  │  CC-04: Vulnerability Management                                        │   │
│  │  CC-05: Incident Response                                               │   │
│  │  CC-06: Change Management (GitOps)                                      │   │
│  │  CC-07: Data Retention & Disposal                                       │   │
│  │  CC-08: Tenant Isolation (RLS, Network)                                 │   │
│  │  CC-09: Business Continuity / DR                                        │   │
│  │  CC-10: Supplier Risk Management                                        │   │
│  │  CC-11: Security Training & Awareness                                   │   │
│  │  CC-12: Physical Security (Cloud Provider)                              │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│           │                    │                    │                    │      │
│           ▼                    ▼                    ▼                    ▼      │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐      ┌────────┐   │
│  │   SOC 2     │      │  ISO 27001  │      │    GDPR     │      │ HIPAA  │   │
│  │  (Trust Svcs)│      │  (Annex A)  │      │  (Art 5-32) │      │ (45 CFR)│   │
│  ├─────────────┤      ├─────────────┤      ├─────────────┤      ├────────┤   │
│  │ CC6.1, CC6.2│      │ A.5.1, A.9.1│      │ Art 5, 25   │      │164.308 │   │
│  │ CC6.3, CC6.4│      │ A.9.2, A.9.3│      │ Art 32      │      │164.310 │   │
│  │ CC6.5, CC6.6│      │ A.9.4, A.8.2│      │ Art 33, 34  │      │164.312 │   │
│  │ CC7.1, CC7.2│      │ A.12.1, A.12.2│    │ Art 28      │      │164.314 │   │
│  │ CC7.3, CC7.4│      │ A.12.4, A.12.6│    │ Art 30      │      │        │   │
│  │ CC8.1, CC9.1│      │ A.13.1, A.13.2│    │ Art 17, 19  │      │        │   │
│  │ CC9.2, A1.1 │      │ A.14.1, A.14.2│    │ Art 25      │      │        │   │
│  │ A1.2, A1.3  │      │ A.16.1, A.17.1│    │ Art 28      │      │        │   │
│  └─────────────┘      └─────────────┘      └─────────────┘      └────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Continuous Compliance Architecture

### 3.1 Compliance Automation Pipeline

```ascii
┌─────────────────────────────────────────────────────────────────────────────────┐
│                  CONTINUOUS COMPLIANCE MONITORING                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  POLICY DEFINITIONS (Policy-as-Code)                                           │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  • Rego/OPA policies for K8s, Terraform, Cloud, App config              │   │
│  │  • SOC 2 / ISO 27001 control mappings                                   │   │
│  │  • Versioned in Git (compliance/policies/)                              │   │
│  │  • Tested in CI pipeline                                                │   │
│  └────────────────────────────────┬────────────────────────────────────────┘   │
│                                   │                                             │
│                                   ▼                                             │
│  EVIDENCE COLLECTION ENGINE                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │   │
│  │  │  INFRASTRUCTURE│  │ APPLICATION │  │  DATA       │  │  PROCESS    │    │   │
│  │  │  (Cloud/    ) │  │  (Code/     ) │  │  (DB/      ) │  │  (GitOps/  ) │    │   │
│  │  │  K8s/Net)    │  │  Config)    │  │  Backups)   │  │  CI/CD)     │    │   │
│  │  │             │  │             │  │             │  │             │    │   │
│  │  │ • AWS Config│  │ • SAST/DAST │  │ • Backup    │  │ • Deploy    │    │   │
│  │  │ • K8s Audit │  │ • SBOM      │  │   reports   │  │   logs      │    │   │
│  │  │ • Cilium    │  │ • Dependabot│  │ • RLS check │  │ • Approval  │    │   │
│  │  │   policies  │  │ • License   │  │ • PII scan  │  │   records   │    │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │   │
│  └────────────────────────────────┬────────────────────────────────────────┘   │
│                                   │                                             │
│                                   ▼                                             │
│  COMPLIANCE STATE STORE (Immutable)                                            │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  • Time-series compliance state (Pass/Fail/Deviation)                   │   │
│  │  • Evidence artifacts (signed, timestamped)                             │   │
│  │  • Drift detection alerts                                               │   │
│  │  • Audit-ready evidence packages                                        │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                   │                                             │
│               ┌───────────────────┼───────────────────┐                        │
│               ▼                   ▼                   ▼                        │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐             │
│  │  DASHBOARDS      │  │  ALERTING        │  │  AUDIT EXPORT    │             │
│  │  (Grafana)       │  │  (PagerDuty/    │  │  (Evidence       │             │
│  │  • Real-time     │  │   Slack)        │  │   packages)      │             │
│  │    compliance    │  │  • Drift alerts │  │  • SOC 2 package │             │
│  │  • Trend         │  │  • Evidence     │  │  • ISO 27001     │             │
│  │  • Evidence      │  │    gaps         │  │  • GDPR records  │             │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘             │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Policy-as-Code Examples

```rego
# compliance/policies/k8s/rbac.rego
package compliance.k8s.rbac

import future.keywords.if
import future.keywords.in

# SOC 2 CC6.1 / ISO 27001 A.9.2 - Least privilege RBAC
deny[msg] {
    input.kind == "RoleBinding"
    input.subjects[_].kind == "User"
    input.subjects[_].name == "system:anonymous"
    msg := "RoleBinding grants access to anonymous user"
}

deny[msg] {
    input.kind == "ClusterRoleBinding"
    input.roleRef.name == "cluster-admin"
    input.subjects[_].kind == "ServiceAccount"
    not trusted_service_account(input.subjects[_])
    msg := sprintf("ClusterRoleBinding grants cluster-admin to untrusted SA: %s", [input.subjects[_].name])
}

trusted_service_account(sa) {
    sa.namespace == "kube-system"
    sa.name == "default"
}

trusted_service_account(sa) {
    sa.namespace == "observability"
    sa.name == "prometheus"
}
```

```rego
# compliance/policies/aws/s3_encryption.rego
package compliance.aws.s3

# SOC 2 CC6.7 / ISO 27001 A.10.1 - Encryption at rest
deny[msg] {
    input.resource_type == "aws_s3_bucket"
    not input.server_side_encryption_configuration
    msg := sprintf("S3 bucket %s lacks server-side encryption", [input.name])
}

deny[msg] {
    input.resource_type == "aws_s3_bucket"
    input.server_side_encryption_configuration.rule[_].apply_server_side_encryption_by_default.sse_algorithm != "AES256"
    input.server_side_encryption_configuration.rule[_].apply_server_side_encryption_by_default.sse_algorithm != "aws:kms"
    msg := sprintf("S3 bucket %s uses non-compliant encryption algorithm", [input.name])
}
```

```yaml
# compliance/policies/terraform/main.tf (validated by OPA in CI)
resource "aws_db_instance" "primary" {
  # SOC 2 CC6.7 / ISO 27001 A.10.1
  storage_encrypted = true
  kms_key_id = aws_kms_key.rds.arn
  
  # SOC 2 CC7.2 / ISO 27001 A.12.4
  backup_retention_period = 35
  copy_tags_to_snapshot = true
  
  # SOC 2 CC6.1 / ISO 27001 A.9.1
  deletion_protection = true
}
```

---

## 4. Evidence Collection & Management

### 4.1 Evidence Categories & Retention

| Evidence Type | Source | Collection Frequency | Retention | Format | Integrity |
|---------------|--------|---------------------|-----------|--------|-----------|
| **Infrastructure Config** | AWS Config, Terraform State | Continuous (drift) / Daily (snapshot) | 7 years | JSON/HCL | SHA-256 + GPG signed |
| **K8s Audit Logs** | K8s API Server → Loki | Real-time | 7 years | JSON | Immutable Loki storage |
| **Application Logs** | App → Loki | Real-time | 7 years | JSON | Immutable Loki storage |
| **Vulnerability Scans** | Trivy, Dependabot, Snyk | Per PR + Daily | 3 years | SARIF/JSON | Signed attestation |
| **Deployment Records** | ArgoCD, GitHub Actions | Per deployment | 7 years | JSON | Git commit signed |
| **Access Reviews** | IAM, K8s RBAC | Quarterly | 3 years | CSV/PDF | Approved by manager |
| **Incident Records** | PagerDuty, Incident Tracker | Per incident | 7 years | JSON/PDF | Immutable |
| **Penetration Test Reports** | External vendor | Annual | 3 years | PDF | Vendor signed |
| **Training Records** | LMS | Quarterly | 3 years | CSV | HR signed |
| **Supplier Assessments** | Vendor risk tool | Annual | 3 years | PDF | Procurement signed |
| **DR Test Results** | Runbook execution | Quarterly | 3 years | PDF | IC signed |
| **Data Processing Records (GDPR Art.30)** | App metadata | Continuous | 7 years | JSON | Automated |

### 4.2 Automated Evidence Collection

```python
# backend/nexusasset/nxa_audit/services/evidence_collector.py
"""
Automated evidence collector for continuous compliance.
Runs as scheduled job (CronJob in K8s) and on-demand for audits.
"""

from dataclasses import dataclass
from datetime import datetime, timedelta
from typing import List, Dict, Any
import hashlib
import gpg
import json


@dataclass
class EvidenceArtifact:
    control_id: str
    framework: str  # soc2, iso27001, gdpr
    evidence_type: str
    title: str
    description: str
    collection_timestamp: datetime
    data: Dict[str, Any]
    hash_sha256: str
    gpg_signature: str
    collector_version: str = "1.0.0"


class EvidenceCollector:
    def __init__(self, gpg_key_id: str):
        self.gpg_key_id = gpg_key_id
        self.artifacts: List[EvidenceArtifact] = []
    
    def collect_all(self, frameworks: List[str] = None) -> List[EvidenceArtifact]:
        """Collect evidence for specified frameworks."""
        frameworks = frameworks or ["soc2", "iso27001", "gdpr"]
        
        # Infrastructure evidence
        self._collect_aws_config()
        self._collect_terraform_state()
        self._collect_k8s_resources()
        
        # Application evidence
        self._collect_sast_results()
        self._collect_sbom()
        self._collect_dependency_scan()
        
        # Data evidence
        self._collect_backup_verification()
        self._collect_rls_verification()
        self._collect_pii_scan()
        
        # Process evidence
        self._collect_deployment_records()
        self._collect_access_reviews()
        self._collect_incident_records()
        
        # GDPR specific
        if "gdpr" in frameworks:
            self._collect_gdpr_records()
        
        return self.artifacts
    
    def _collect_aws_config(self):
        """Collect AWS Config compliance snapshots."""
        # Query AWS Config for compliance status
        # Store as evidence artifact
        pass
    
    def _collect_k8s_resources(self):
        """Collect K8s resource configurations for RBAC, NetworkPolicy, etc."""
        # Query K8s API for all resources
        # Validate against OPA policies
        # Store compliant/non-compliant resources
        pass
    
    def _collect_backup_verification(self):
        """Collect database backup verification evidence."""
        # Verify latest backup exists, is restorable, encrypted
        # Store verification result
        pass
    
    def _collect_rls_verification(self):
        """Collect RLS policy verification for tenant isolation."""
        # Query pg_policies for all tenant schemas
        # Verify policies exist and are correct
        pass
    
    def _create_artifact(self, control_id: str, framework: str, evidence_type: str,
                         title: str, description: str, data: Dict) -> EvidenceArtifact:
        """Create signed evidence artifact."""
        timestamp = datetime.utcnow()
        data_json = json.dumps(data, sort_keys=True, default=str)
        hash_sha256 = hashlib.sha256(data_json.encode()).hexdigest()
        
        # GPG sign
        gpg_signature = self._gpg_sign(data_json)
        
        artifact = EvidenceArtifact(
            control_id=control_id,
            framework=framework,
            evidence_type=evidence_type,
            title=title,
            description=description,
            collection_timestamp=timestamp,
            data=data,
            hash_sha256=hash_sha256,
            gpg_signature=gpg_signature,
        )
        
        self.artifacts.append(artifact)
        return artifact
    
    def _gpg_sign(self, data: str) -> str:
        """GPG sign evidence data."""
        # Implementation uses python-gnupg
        pass
    
    def export_audit_package(self, framework: str, output_dir: str):
        """Export audit-ready evidence package."""
        import zipfile
        from pathlib import Path
        
        framework_artifacts = [a for a in self.artifacts if a.framework == framework]
        
        package_path = Path(output_dir) / f"audit_package_{framework}_{datetime.utcnow().strftime('%Y%m%d')}.zip"
        
        with zipfile.ZipFile(package_path, 'w', zipfile.ZIP_DEFLATED) as zf:
            # Manifest
            manifest = {
                "framework": framework,
                "generated_at": datetime.utcnow().isoformat(),
                "collector_version": "1.0.0",
                "artifacts": [
                    {
                        "control_id": a.control_id,
                        "evidence_type": a.evidence_type,
                        "title": a.title,
                        "timestamp": a.collection_timestamp.isoformat(),
                        "hash_sha256": a.hash_sha256,
                        "gpg_signature": a.gpg_signature,
                    }
                    for a in framework_artifacts
                ]
            }
            zf.writestr("MANIFEST.json", json.dumps(manifest, indent=2))
            
            # Individual artifacts
            for artifact in framework_artifacts:
                filename = f"{artifact.control_id}_{artifact.evidence_type}.json"
                zf.writestr(filename, json.dumps(artifact.data, indent=2, default=str))
        
        return package_path
```

---

## 5. SOC 2 Type II Compliance

### 5.1 Trust Services Criteria Mapping

| TSC | Criteria | NexusAsset Control | Evidence | Test Frequency |
|-----|----------|-------------------|----------|----------------|
| **CC6.1** | Logical access | RBAC/ABAC, MFA, JIT access | IAM policies, K8s RBAC, access reviews | Quarterly |
| **CC6.2** | Credential management | Password policy, key rotation, Vault | Vault audit logs, rotation records | Monthly |
| **CC6.3** | Network segmentation | VPC, Cilium NetworkPolicies, mTLS | NetworkPolicy configs, flow logs | Continuous |
| **CC6.4** | Data classification | Data labeling, encryption, DLP | Data catalog, encryption config | Quarterly |
| **CC6.5** | Vulnerability management | SAST/DAST/SCA, patch management | Scan reports, patch deployment logs | Continuous |
| **CC6.6** | Malware protection | Container scanning, runtime protection | Trivy/Falco alerts, image signatures | Continuous |
| **CC7.1** | System monitoring | Prometheus/Grafana/Loki/Tempo | Dashboard configs, alert rules | Continuous |
| **CC7.2** | Incident response | Runbooks, PagerDuty, postmortems | Incident records, postmortem docs | Per incident |
| **CC7.3** | Root cause analysis | 5 Whys, blameless postmortems | Postmortem documents | Per SEV-1/2 |
| **CC7.4** | Response improvement | Action item tracking, runbook updates | Action item tracker, runbook versions | Quarterly |
| **CC8.1** | Change management | GitOps, PR reviews, automated tests | Deployment records, PR approvals | Per deployment |
| **CC9.1** | Risk assessment | Annual risk assessment, threat modeling | Risk register, threat models | Annual |
| **A1.1** | Availability monitoring | SLOs, error budgets, synthetic monitoring | SLO dashboards, burn rate alerts | Continuous |
| **A1.2** | Capacity management | Auto-scaling, capacity planning | Scaling events, capacity reports | Monthly |
| **A1.3** | Backup & recovery | Automated backups, DR tests | Backup verification, DR test results | Quarterly |

### 5.2 SOC 2 Evidence Package Structure

```
soc2-evidence-package-2026-11-09/
├── MANIFEST.json
├── CC6.1_Access_Control/
│   ├── iam_policies.json
│   ├── k8s_rbac_bindings.json
│   ├── access_review_2026_Q3.csv
│   ├── mfa_enforcement_report.json
│   └── jit_access_logs.json
├── CC6.2_Credential_Management/
│   ├── vault_audit_logs.json
│   ├── key_rotation_schedule.json
│   ├── rotated_keys_2026_Q3.json
│   └── password_policy_config.json
├── CC6.3_Network_Segmentation/
│   ├── cilium_network_policies.yaml
│   ├── vpc_flow_logs_sample.json
│   ├── mtls_verification.json
│   └── waf_rules.json
├── CC7.1_System_Monitoring/
│   ├── prometheus_rules.yaml
│   ├── grafana_dashboards.json
│   ├── slo_definitions.yaml
│   └── alert_routing_config.yaml
├── CC7.2_Incident_Response/
│   ├── incident_records_2026_Q3.json
│   ├── postmortem_INC-2026-045.pdf
│   ├── runbook_RB-003.md
│   └── fire_drill_results_Q3.pdf
└── ... (all criteria)
```

---

## 6. ISO 27001:2022 Compliance

### 6.1 Annex A Control Implementation

| Control | Title | Implementation | Evidence |
|---------|-------|----------------|----------|
| **A.5.1** | Information security policies | Policy repo, annual review | Policy documents, approval records |
| **A.5.2** | Information security roles | RACI matrix, role definitions | Org chart, role assignments |
| **A.6.1** | Internal organization | Segregation of duties, on-call | On-call schedule, access matrix |
| **A.7.1** | Screening | Background checks, NDAs | HR records, signed NDAs |
| **A.7.2** | Terms of employment | Security clauses in contracts | Employment agreements |
| **A.8.1** | Asset inventory | CMDB (auto-discovered), tagging | Asset inventory reports |
| **A.8.2** | Information classification | Data classification schema, labeling | Classification policy, tagged resources |
| **A.8.3** | Media handling | Encryption, secure disposal | Encryption configs, disposal certificates |
| **A.9.1** | Access control policy | RBAC/ABAC policy, least privilege | Policy doc, access control matrix |
| **A.9.2** | User access management | Provisioning/deprovisioning, reviews | IAM logs, quarterly access reviews |
| **A.9.3** | User responsibilities | Acceptable use, security training | Training records, acknowledgments |
| **A.9.4** | System/application access | MFA, JIT, session management | Auth logs, session configs |
| **A.10.1** | Cryptographic controls | AES-256, TLS 1.3, KMS | Encryption configs, key hierarchy |
| **A.12.1** | Operational procedures | Runbooks, SOPs, automation | Runbook library, execution logs |
| **A.12.2** | Change management | GitOps, CI/CD, approval gates | Deployment records, PR approvals |
| **A.12.4** | Event logging | Centralized logging, retention | Loki config, retention policies |
| **A.12.6** | Technical vulnerability mgmt | SAST/DAST/SCA, patching | Scan reports, patch deployment |
| **A.13.1** | Network security | Network policies, mTLS, WAF | Cilium policies, WAF rules |
| **A.13.2** | Information transfer | API security, encryption in transit | API specs, TLS configs |
| **A.14.1** | Secure development | SDLC gates, threat modeling | Pipeline config, threat models |
| **A.14.2** | Security testing | SAST/DAST/IAST, pen testing | Scan results, pentest reports |
| **A.16.1** | Incident management | Runbooks, escalation, comms | Incident records, postmortems |
| **A.17.1** | Business continuity | DR plan, RTO/RPO, testing | DR plan, test results |

### 6.2 Statement of Applicability (SoA)

```markdown
# ISO 27001 Statement of Applicability - NexusAsset Cloud

**Version:** 1.0 | **Date:** 2026-11-09 | **Owner:** Security & Compliance

## Scope
Information Security Management System (ISMS) for NexusAsset Cloud SaaS platform
including all production environments, customer data, and supporting processes.

## Control Selection
All 93 Annex A controls are **applicable** and **implemented**.
No exclusions.

## Control Implementation Status

| Control | Status | Implementation | Evidence Location |
|---------|--------|----------------|-------------------|
| A.5.1 | ✅ Implemented | Policy-as-code, annual review | compliance/policies/ |
| A.5.2 | ✅ Implemented | RACI in Confluence, role bindings | compliance/rbac/ |
| A.6.1 | ✅ Implemented | On-call rotation, segregated duties | ops/oncall/ |
| ... | ... | ... | ... |

## Risk Treatment
All risks from risk assessment (DOC-02) have treatments mapped to controls.
Residual risk accepted per risk register.
```

---

## 7. GDPR Compliance

### 7.1 Lawful Basis & Data Processing

| Data Category | Lawful Basis (Art. 6) | Purpose | Retention | Controller/Processor |
|---------------|----------------------|---------|-----------|---------------------|
| **Tenant admin data** | Contract (b) | Account management, billing | Contract + 7 years | Controller |
| **Asset inventory data** | Legitimate interest (f) | ITAM, security, compliance | Tenant-defined (min 1 year) | Processor |
| **Sync data (Lansweeper)** | Contract (b) | Asset discovery, reconciliation | Sync cycle + 90 days | Processor |
| **Audit logs** | Legal obligation (c) / Legitimate interest (f) | Security, compliance, debugging | 7 years (SOC 2) / 13 months (ops) | Controller |
| **Usage analytics** | Consent (a) | Product improvement | 26 months | Controller |
| **Support tickets** | Contract (b) | Customer support | Resolution + 3 years | Controller |

### 7.2 Data Subject Rights Implementation

| Right (Art.) | Implementation | API Endpoint | SLA |
|--------------|----------------|--------------|-----|
| **Access (15)** | Self-service portal + API | `GET /api/v1/me/data-export` | 30 days |
| **Rectification (16)** | Self-service + Support | `PATCH /api/v1/me/profile` | 30 days |
| **Erasure (17)** | Automated workflow (DOC-14) | `DELETE /api/v1/me` (admin) | 30 days |
| **Restriction (18)** | Processing flag | `POST /api/v1/me/restrict-processing` | 30 days |
| **Portability (20)** | Standardized export (JSON/CSV) | `GET /api/v1/me/data-export?format=csv` | 30 days |
| **Objection (21)** | Opt-out flags, marketing suppression | `POST /api/v1/me/opt-out` | Immediate |

### 7.3 GDPR Technical Measures

```python
# backend/nexusasset/nxa_core/compliance/gdpr.py

class GDPRCompliance:
    """GDPR compliance utilities."""
    
    @staticmethod
    def anonymize_personal_data(tenant_id: str, user_id: str = None):
        """
        Anonymize personal data per Art. 17 (Right to Erasure).
        Called by tenant deprovisioning or user deletion.
        """
        from django.db import connection
        
        with connection.cursor() as cursor:
            # Set search path to tenant schema
            cursor.execute(f'SET search_path TO "tenant_{tenant_id.replace("-", "_")}", public')
            
            # Anonymize user data (retain audit trail without PII)
            if user_id:
                cursor.execute("""
                    UPDATE nxa_core_user
                    SET email = 'anonymized_' || id || '@example.com',
                        first_name = 'Anonymized',
                        last_name = 'User',
                        phone = NULL,
                        avatar_url = NULL,
                        last_login = NULL,
                        is_active = FALSE
                    WHERE id = %s
                """, [user_id])
            
            # Anonymize audit logs (keep structure, remove PII)
            cursor.execute("""
                UPDATE nxa_audit_auditlog
                SET actor_email = 'anonymized_' || actor_id || '@example.com',
                    metadata = jsonb_set(
                        metadata,
                        '{actor_name}',
                        '"Anonymized User"'
                    )
                WHERE actor_id = %s
            """, [user_id])
            
            # Remove from notification preferences
            cursor.execute("""
                DELETE FROM nxa_notifications_preference
                WHERE user_id = %s
            """, [user_id])
    
    @staticmethod
    def export_user_data(tenant_id: str, user_id: str) -> Dict:
        """
        Export all user data per Art. 15 (Right of Access) and Art. 20 (Portability).
        Returns structured JSON for download.
        """
        # Implementation collects data from all tenant tables
        pass
    
    @staticmethod
    def record_processing_activity(tenant_id: str, activity: Dict):
        """
        Record processing activity per Art. 30 (Records of Processing Activities).
        Automated: called by data processing pipelines.
        """
        from nxa_audit.models import ProcessingActivity
        
        ProcessingActivity.objects.create(
            tenant_id=tenant_id,
            activity_type=activity['type'],  # collection, storage, transfer, deletion
            data_categories=activity['categories'],
            purpose=activity['purpose'],
            lawful_basis=activity['lawful_basis'],
            recipients=activity.get('recipients', []),
            transfer_mechanism=activity.get('transfer_mechanism'),
            retention_period=activity['retention'],
            technical_measures=activity['technical_measures'],
            organizational_measures=activity['organizational_measures'],
        )
```

### 7.4 Data Processing Agreement (DPA)

```markdown
# Data Processing Agreement - NexusAsset Cloud

**Version:** 2.0 | **Effective:** 2026-01-01

## 1. Parties
- **Controller:** Customer (Tenant)
- **Processor:** NexusAsset Cloud Ltd.

## 2. Processing Details
- **Subject matter:** IT Asset Management, Software License Management, Vulnerability Management
- **Duration:** Term of agreement + 90 days
- **Nature:** Collection, storage, analysis, reporting, synchronization
- **Purpose:** Provide SaaS services per MSA
- **Categories:** Asset data, software inventory, vulnerability data, user access data
- **Data subjects:** Customer employees, contractors, assets

## 3. Processor Obligations
- Process only on documented instructions
- Ensure confidentiality commitments
- Implement Art. 32 security measures (this DOC-16)
- Sub-processor management (listed in Annex 1)
- Data subject rights assistance
- Breach notification within 24 hours
- Deletion/return of data on termination
- Audit cooperation (SOC 2 reports provided)

## 4. Technical & Organizational Measures
See Annex 2: Security Measures (aligned with DOC-12)

## 5. Sub-processors
- AWS (hosting) - EU/US regions per tenant choice
- Cloudflare (WAF/CDN) - Global
- PagerDuty (incident mgmt) - US (SCC)
- SendGrid (email) - EU/US
```

---

## 8. Audit Readiness Procedures

### 8.1 Audit Preparation Timeline

| Timeline | Activity | Owner | Deliverable |
|----------|----------|-------|-------------|
| **T-90 days** | Audit kickoff, scope confirmation | Compliance Lead | Audit plan, team assignments |
| **T-60 days** | Evidence gap analysis | Compliance Team | Gap remediation plan |
| **T-45 days** | Automated evidence collection run | Platform/Engineering | Evidence package v1 |
| **T-30 days** | Manual evidence gathering | All teams | Complete evidence package |
| **T-21 days** | Internal readiness review | Compliance + Legal | Readiness sign-off |
| **T-14 days** | Auditor access provisioning | Platform | Read-only auditor accounts |
| **T-7 days** | Final evidence package delivery | Compliance | Audit package (encrypted) |
| **T-0** | Audit fieldwork begins | All | — |
| **T+30 days** | Audit findings review | Compliance + Leadership | Remediation plan |
| **T+90 days** | Remediation complete | All teams | Evidence of remediation |

### 8.2 Auditor Access Provisioning

```bash
# Create auditor read-only access (valid for audit duration)
# 1. AWS: Create IAM role with SecurityAudit policy
aws iam create-role --role-name AuditorReadOnly \
  --assume-role-policy-document file://auditor-trust-policy.json

aws iam attach-role-policy --role-name AuditorReadOnly \
  --policy-arn arn:aws:iam::aws:policy/SecurityAudit

# 2. K8s: Create read-only ClusterRole
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: auditor-read-only
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets", "nodes"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets", "daemonsets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["networking.k8s.io"]
  resources: ["networkpolicies", "ingresses"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["cilium.io"]
  resources: ["ciliumnetworkpolicies"]
  verbs: ["get", "list", "watch"]
EOF

# 3. Create ServiceAccount and bind
kubectl create serviceaccount auditor -n audit
kubectl create clusterrolebinding auditor-binding \
  --clusterrole=auditor-read-only \
  --serviceaccount=audit:auditor

# 4. Generate token (valid 90 days)
kubectl create token auditor -n audit --duration=2160h > auditor_token.txt

# 5. Provide to auditor via secure channel (1Password, encrypted email)
```

### 8.3 Evidence Package Delivery

```bash
# Generate and deliver audit package
# 1. Run collector
python -m nxa_audit.services.evidence_collector \
  --frameworks soc2 iso27001 gdpr \
  --output-dir /tmp/audit_package_20261109

# 2. Verify integrity
sha256sum /tmp/audit_package_20261109/*.zip > checksums.txt
gpg --sign --armor --output checksums.txt.asc checksums.txt

# 3. Encrypt for auditor (using their PGP key)
gpg --encrypt --recipient auditor@big4.com \
  --output audit_package_20261109_soc2.zip.gpg \
  /tmp/audit_package_20261109/soc2-evidence-package-2026-11-09.zip

# 4. Deliver via secure file transfer (SFTP, 1Password, or secure email)
# 5. Provide checksums and verification instructions
```

---

## 9. Compliance Metrics & KPIs

| Metric | Target | Measurement | Frequency |
|--------|--------|-------------|-----------|
| **Policy Compliance Rate** | 100% (critical), >95% (all) | OPA policy evaluation | Continuous |
| **Evidence Freshness** | < 24 hours | Last collection timestamp | Daily |
| **Drift Detection Time** | < 1 hour | Time from drift to alert | Continuous |
| **Drift Remediation Time** | < 24 hours (critical), < 7 days (all) | Ticket resolution time | Monthly |
| **Access Review Completion** | 100% quarterly | Review records | Quarterly |
| **Training Completion** | 100% annual | LMS records | Annual |
| **Incident Response Time** | Per SLO (DOC-13) | PagerDuty analytics | Per incident |
| **Audit Findings (Critical)** | 0 | Audit reports | Annual |
| **Audit Findings (High)** | < 3 | Audit reports | Annual |
| **Remediation Time (Audit)** | < 90 days | Tracking system | Per finding |

---

## 10. Compliance Dashboard (Grafana)

```json
{
  "dashboard": "compliance-overview",
  "panels": [
    {
      "title": "Policy Compliance Score",
      "type": "gauge",
      "targets": [
        {"expr": "avg(compliance_policy_compliance_rate{severity=\"critical\"})", "legendFormat": "Critical"},
        {"expr": "avg(compliance_policy_compliance_rate{severity=\"high\"})", "legendFormat": "High"},
        {"expr": "avg(compliance_policy_compliance_rate{severity=\"medium\"})", "legendFormat": "Medium"}
      ]
    },
    {
      "title": "Evidence Freshness",
      "type": "table",
      "targets": [
        {"expr": "compliance_evidence_age_hours", "format": "table"}
      ]
    },
    {
      "title": "Open Drift Violations",
      "type": "stat",
      "targets": [
        {"expr": "count(compliance_drift_violations{status=\"open\"})", "legendFormat": "Open Drifts"}
      ]
    },
    {
      "title": "Access Review Status",
      "type": "piechart",
      "targets": [
        {"expr": "count(compliance_access_review{status=\"completed\"})", "legendFormat": "Completed"},
        {"expr": "count(compliance_access_review{status=\"pending\"})", "legendFormat": "Pending"},
        {"expr": "count(compliance_access_review{status=\"overdue\"})", "legendFormat": "Overdue"}
      ]
    },
    {
      "title": "Training Completion",
      "type": "gauge",
      "targets": [
        {"expr": "compliance_training_completion_rate", "legendFormat": "Completion %"}
      ]
    }
  ]
}
```

---

## 11. Future Compliance Roadmap

| Quarter | Initiative | Frameworks | Description |
|---------|------------|------------|-------------|
| **Q1 2027** | **FedRAMP Medium Readiness** | NIST 800-53, FedRAMP | Gap analysis, control implementation, 3PAO engagement |
| **Q2 2027** | **HIPAA Compliance** | HIPAA | BAA template, PHI handling controls, breach notification |
| **Q3 2027** | **PCI DSS SAQ A** | PCI DSS | If payment processing added, scope reduction via tokenization |
| **Q4 2027** | **ISO 27701 (PIMS)** | GDPR enhancement | Privacy Information Management System certification |
| **2028** | **SOC 3 Report** | SOC 2 | Public-facing trust report for marketing |

---

## 12. References

| Doc ID | Title | Link |
|--------|-------|------|
| DOC-04 | Domain Model | [04-domain-model.md](../00-overview/04-domain-model.md) |
| DOC-06 | Infrastructure Architecture | [06-infrastructure-architecture.md](../00-overview/06-infrastructure-architecture.md) |
| DOC-12 | Security Architecture | [12-security-architecture.md](../00-overview/12-security-architecture.md) |
| DOC-13 | Observability | [13-observability.md](../00-overview/13-observability.md) |
| DOC-14 | Data Migration | [14-data-migration-seed-strategy.md](../00-overview/14-data-migration-seed-strategy.md) |
| DOC-15 | Operations Runbook | [15-operations-runbook.md](../00-overview/15-operations-runbook.md) |
| ADR-037 | Compliance Framework Selection | [ADR-037](../14-appendices/adr-index.md#adr-037) |
| ADR-038 | Evidence Collection Automation | [ADR-038](../14-appendices/adr-index.md#adr-038) |
| ADR-039 | GDPR Implementation | [ADR-039](../14-appendices/adr-index.md#adr-039) |
| ADR-040 | Policy-as-Code | [ADR-040](../14-appendices/adr-index.md#adr-040) |
| ADR-041 | Audit Readiness | [ADR-041](../14-appendices/adr-index.md#adr-041) |
| ADR-042 | Continuous Compliance | [ADR-042](../14-appendices/adr-index.md#adr-042) |

---

> **Document Control:** This document is maintained by Security & Compliance. Changes require review by CISO and Legal. Audit evidence packages generated per schedule. Next review: 2027-02-09.