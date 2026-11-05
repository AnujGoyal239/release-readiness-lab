# OrderFlow

This repository contains the Release Readiness Investigation & Deployment Approval Lab for **OrderFlow v2.3.0**.

## Application

A Node.js Express microservice responsible for order routing and flow processing.

### Endpoints
- `GET /`: Health & welcome message
- `GET /health`: Microservice health check & version reporting

## Current Release Candidate
- **Version:** `v2.3.0`
- **Release Status:** Production-Ready (Release Readiness Pack Complete)
- **Deployment Strategy:** Kubernetes Zero-Downtime RollingUpdate (`maxSurge: 1`, `maxUnavailable: 0`)
- **Rollback Strategy:** Automated `scripts/rollback.sh` via Kubernetes rollout undo

## Operational Documentation
- [Release Readiness Pack (RRP)](./release/RELEASE_READINESS_PACK.md)
- [Release Metadata](./release/release.json)
- [Approvals Policy](./release/approvals.yml)
- [Deployment Configuration](./release/deployment-config.yml)
- [Rollback Plan](./release/rollback.yml)
- [Release Summary](./release/release-summary.yml)

## Getting Started

### Local Installation & Testing
```bash
cd app
npm install
npm test
npm start
```

### Deployment Pipeline
The CI/CD pipeline is defined in `.github/workflows/release.yml`. It enforces:
1. `validate-release`: Unit testing, version verification, and manifest existence checks.
2. `deploy-production`: Gated release verification executed only after successful validation.
