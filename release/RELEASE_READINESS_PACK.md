# OrderFlow v2.3.0 - Final Release Readiness Pack (RRP)

**Release Candidate:** OrderFlow v2.3.0  
**Target Environment:** Production (`production` namespace)  
**Release Engineer:** Anuj Goyal  
**Date:** 2026-11-05  
**Release Status:** GO (Approved for Production Deployment)

---

## 1. Release Version Information
- **Application Name:** OrderFlow
- **Release Version:** `2.3.0`
- **Previous Stable Release:** `2.2.0`
- **Docker Image Tag:** `lab-app:2.3.0`
- **Git Branch:** `investigation/release-readiness` -> `main`
- **Commit Reference:** Monitored via Git SHA & semantic tagging
- **Release Type:** Minor feature update & stability hardening
- **Release Metadata Manifest:** `release/release.json` (Validated against strict schema)

---

## 2. Deployment Strategy
- **Deployment Type:** Kubernetes Rolling Update (`RollingUpdate`)
- **Zero-Downtime Parameters:**
  - `maxSurge: 1` (Allows 1 temporary additional pod during rollover)
  - `maxUnavailable: 0` (Guarantees 100% capacity during entire deployment)
- **Replica Count:** 3 pods distributed across cluster worker nodes
- **Health Checks & Probes:**
  - **Readiness Probe:** HTTP GET `/` on port 3000, `initialDelaySeconds: 10`, `periodSeconds: 10`. Ensures traffic is only routed to pods that are completely initialized.
  - **Liveness Probe:** HTTP GET `/` on port 3000, `initialDelaySeconds: 20`, `periodSeconds: 20`. Automatically restarts hung or deadlocked containers.
- **Resource Isolation:**
  - CPU: Request `100m`, Limit `500m`
  - Memory: Request `128Mi`, Limit `512Mi`

---

## 3. Rollback Strategy
- **Rollback Mechanism:** Automated Kubernetes rollout undo (`scripts/rollback.sh`)
- **Rollback Target Version:** Previous stable deployment (`v2.2.0` / previous revision)
- **Automated Trigger Conditions:**
  - Synthetic health check failure (`/` or `/health` returning non-200)
  - Application crash-looping or startup timeouts (`CrashLoopBackOff`)
  - HTTP 5xx error rate spiking above 1% in first 10 minutes post-release
  - Elevated p99 latency exceeding SLO (>500ms)
- **Rollback Procedure:**
  1. Trigger automated rollback script:
     ```bash
     bash scripts/rollback.sh
     ```
  2. The script verifies `kubectl` connectivity and issues:
     ```bash
     kubectl rollout undo deployment/lab-app -n production
     kubectl rollout status deployment/lab-app -n production --timeout=180s
     ```
  3. Verify application health on all 3 pods.
  4. Automatically alert on-call SRE and notify Release Manager.

---

## 4. Deployment Infrastructure Decision
- **Orchestration Platform:** Kubernetes (EKS / GKE production cluster)
- **Namespace:** Dedicated `production` namespace with RBAC and NetworkPolicies
- **Service Networking:**
  - Service Type: `ClusterIP` (Private internal cluster network)
  - Port Mapping: Port 80 -> TargetPort 3000
  - Architectural Rationale: Replaced high-risk public `LoadBalancer` with internal `ClusterIP`. Ingress traffic is managed through a secure API Gateway with TLS termination and WAF protection, preventing direct internet exposure of raw backend pods.
- **Infrastructure as Code (Terraform):**
  - AWS Secret keys sanitized; `aws_secret_key` marked `sensitive = true`, `nullable = false` with hardcoded defaults removed.
- **Secret Management:**
  - All plain-text passwords and API keys removed from manifests.
  - Injected securely at runtime via Kubernetes Secret (`orderflow-production-secrets`):
    - `API_KEY`
    - `DATABASE_PASSWORD`
    - `TOKEN`

---

## 5. Production Approval Checklist

| Verification Gate | Requirement | Status | Owner |
| :--- | :--- | :--- | :--- |
| **Pipeline Sequencing** | Validation stages run strictly prior to production gates | Verified | Release Engineer |
| **Code Stability** | All unit tests and application tests execute clean (0 errors) | Passed | QA Team |
| **Version Consistency** | All manifests, scripts, and release metadata reference v2.3.0 | Verified | DevOps Team |
| **Secret Sanitization** | Zero plain-text credentials in repository or commit history | Passed | SecOps |
| **Network Protection** | Service configured as internal ClusterIP; ingress isolated | Verified | Network / Infra |
| **Rollback Preparedness** | Automated rollback script tested with rollout timeout | Verified | SRE Lead |
| **Sign-Off Policy** | 1 required reviewer, prevention of self-review | Approved | Release Manager |

---

## 6. Deployment Validation Summary
- **CI/CD Pipeline:** GitHub Actions Release Pipeline (`.github/workflows/release.yml`)
- **Stage 1 - Validate Release:**
  - Node.js 20 environment configured.
  - Dependencies installed (`npm install --prefix app`).
  - Unit tests executed (`npm test --prefix app`).
  - Strict release version validation (`release/release.json` Version === '2.3.0').
  - Release manifest integrity check passed (`deployment.yaml`, `service.yaml`, `approvals.yml`, `deployment-config.yml`, `rollback.yml`).
- **Stage 2 - Production Release Gate:**
  - Stage sequencing enforced (`needs: validate-release`).
  - Safe deployment verification executed (`scripts/deploy.sh 2.3.0`).
  - Gate confirmation: Release candidate 2.3.0 verified and approved for production.
- **Final Recommendation:** **GO FOR PRODUCTION RELEASE**
