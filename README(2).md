# DevOps System Design – UI, API & Database (AWS)

## Assignment Summary
This document presents a cloud-native design for a 3-tier application hosted on AWS. The design focuses on deployment, security, scalability, and reliability for the UI (frontend), API (backend), and relational database. A CI/CD workflow is described to automate build, test, and deployment processes.

---

## Architecture Overview
Components:
- **Frontend:** React (static) hosted on **S3 + CloudFront**.
- **Backend:** Containerized services on **ECS Fargate** behind an **Application Load Balancer (ALB)**.
- **Database:** **Amazon RDS (MySQL)** with Multi-AZ and Read Replicas.
- **Secrets:** **AWS Secrets Manager** for credentials.
- **CI/CD:** **GitHub Actions** implementing build → test → deploy pipelines.
- **Monitoring & Observability:** **CloudWatch**, **AWS X-Ray** (optional), and centralized logging (CloudWatch Logs / OpenSearch).

---

## 1. Frontend (UI) Design

### Hosting & Delivery
- Build artifacts uploaded to **S3** (private bucket).
- **CloudFront** distribution serves assets globally and handles TLS termination via **ACM** certificate.
- **Origin Access Control (OAC)** restricts S3 access to CloudFront only.

### Performance & Scaling
- CloudFront caches static assets at edge locations → low latency and reduced origin load.
- S3 + CloudFront provide effectively unlimited horizontal scale.

### Security & Availability
- HTTPS enforced; WAF (AWS WAF) filters malicious traffic.
- S3 lifecycle rules and versioning for asset management.
- Cross-region CloudFront edge locations improve availability.

---

## 2. API (Backend) Design

### Runtime & Deployment
- **ECS Fargate** for serverless container execution; alternative: **EKS** if orchestration complexity grows.
- API sits behind an **ALB** which performs SSL termination and path-based routing.

### Scaling & Resilience
- **Task Auto Scaling** scales ECS tasks based on CPU, memory, or custom CloudWatch metrics.
- ALB health checks ensure only healthy tasks receive traffic.
- Deploy using blue/green or canary strategies (AWS CodeDeploy integration or ECS rolling updates).

### Secrets & Configuration
- Store DB credentials and third-party keys in **Secrets Manager**.
- ECS Task Role grants minimal permissions; use IAM policy scoping.

### Networking & Security
- ECS tasks run in **private subnets** within VPC.
- ALB in public subnets routes requests to private tasks.
- Security Groups:
  - ALB SG: allows HTTPS (443) from the internet.
  - ECS SG: allows inbound from ALB SG only.
  - RDS SG: allows inbound from ECS SG only.

---

## 3. Database Design (RDS MySQL)

### High Availability & Durability
- **Multi-AZ** primary for synchronous replication and automatic failover.
- Enable **Automated Backups** and **Point-in-Time Recovery (PITR)**.

### Scaling
- **Vertical scaling**: upgrade instance class when needed.
- **Horizontal scaling**: create **Read Replicas** for read-heavy workloads and analytics.

### Backup & DR Strategy
- Automated daily snapshots + retention policy (7–35 days).
- Manual snapshots for major releases.
- Consider cross-region read replica or snapshots for disaster recovery.

### Schema Migrations
- Use **Flyway** or **Liquibase** for versioned DB migrations.
- Run migrations in CI with a migration-check stage against a staging database.

---

## 4. CI/CD Design (GitHub Actions - Conceptual)

### Triggers
- Push to `main` or merge to `main`
- Pull Request (PR) for feature branches
- Tag-based release for production

### Stages
1. **Build**
   - Frontend: `npm ci && npm run build`
   - Backend: `npm ci && npm test`
2. **Test**
   - Unit tests, lint, integration smoke tests (use test containers or mock services)
3. **Deploy**
   - Frontend: upload build to S3 and invalidate CloudFront cache
   - Backend: build container image, push to ECR, update ECS task definition (rolling update)
4. **Verification**
   - Automated smoke tests, ALB health checks, CloudWatch alarms
5. **Promotion**
   - Manual approval step for production promotion (protect `main` with branch protections)

---

## 5. Observability & Security Monitoring
- **CloudWatch Dashboards** for latency, error rates, CPU/memory utilization.
- **CloudWatch Logs** for centralized logging; set retention and log group structure.
- Use **AWS Config** and **GuardDuty** for security posture monitoring.

---

---

## Git Commands to create the repo and push files
Run these locally after creating a repo named `devops-design-assignment` on GitHub:

```bash
git clone https://github.com/YOUR_USERNAME/devops-design-assignment.git
cd devops-design-assignment
cp /path/to/README.md .
cp /path/to/diagram.pdf .
cp /path/to/diagram.drawio .
git add README.md diagram.pdf diagram.drawio
git commit -m "Add DevOps design README and architecture diagram"
git push origin main
```


