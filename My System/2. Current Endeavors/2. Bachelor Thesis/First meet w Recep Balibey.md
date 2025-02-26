- **Code Phase:**
    - SAST + Secrets Scanning (Semgrep, GitLeaks, SonarQube)
    - Dependency Scanning (Snyk, Trivy)
- **Build Phase:**
    - Container Scanning (Trivy, Clair)
    - SBOM Generation & Signing (Cosign, SLSA)
- **CI/CD Security:**
    - Infrastructure as Code Scanning (tfsec)
    - Secure Pipeline Configurations
- **Deploy Phase:**
    - Kubernetes Policy Enforcement (OPA/Gatekeeper, Kyverno)
    - Cloud Security Scanning (Prowler, AWS GuardDuty)
- **Runtime Security:**
    - Runtime Threat Detection (Falco, Sysdig)
    - SIEM & Logging (Elastic Security, Grafana Loki)

Points:
- Istio
- DAST tools: OWASP ZAP, AWS CloudTrail, penetration testing
- Compare free vs paid tools
- Interview software devs and ask them about their CI/CD pipelines

Risks:
1. Imperfect tools
2. Not to develop an overly complex project
3. Past incidents in CI/CD pipelines: try to solve them in the thesis
4. 3rd party libraries, images: are the tools really effective for all cases?
5. False alerts? Validation of security findings
6. Create reports using LLM: make LLM summarize/report the vulnerabilities

****

A lot of rant on the usefulness of DevSecOps
Test: what does the development speed & security look like with and without each tool? Simulate different attacks to test the successfulness of each individual DevSecOps practice.

Design Stage:
- Threat modelling

Development Stage:
- Precommit hooks
- Repository Access Control

Build Stage:
- SAST
- SCA
- Secret Management

Test (Continuous deployment, acceptance stage):
- DAST tools, comprehensive fuzzing, ZAP scan, penetration testing (BreachLock?)

Production, post-deployment:
- Logging, Monitoring & Observability, Alerting
- Runtime defense (RASP)

+: Access Controls, Secret Management
+: Results from DAST and SAST can be compared to weed out false-positives
+: Compliance? (e.g. PCI DSS)
+: Open Policy Agent/Gatekeeper, Kyverno?
+: IAST?

At the end: case studies of successful DevSecOps stories; interview a senior dev who implemented DevSecOps and ask about their experience.

# DevSecOps
#### 1. Design (Threat Modeling)

Identify security risks and regulatory compliance requirements.

- Perform Threat Modeling using **OWASP Threat Dragon** to map attack vectors
- Define security baselines and compliance needs for an ecommerce platform using PCI DSS and NIST

#### 2. Develop (Precommit hook & Repository Access Control)

Enforce secure coding standards and access control mechanisms.

- Use pre-commit hooks with **Husky** and enforce secure coding best practices through a local linter **ESLint for JS** and **GolangCI-Lint**
- Include repository protection on GitHub with **GitHub branch rules**

#### 3. Build (SAST, SCA, Secret Management)

Validates the security of the application code, dependencies, and infrastructure.

- Perform Static Application Security Testing (SAST) to detect static code vulnerabilities with **SonarQube**
- Perform Software Composition Analysis (SCA) to identify outdated libraries/dependencies with **Snyk**
- In case infrastructure changes are detected, scan Infrastructure as Code (IaC) configurations for security misconfigurations using **Checkov**
- Implement secrets management through **AWS Secrets Manager**

#### 4. Acceptance Testing (DAST, Penetration Testing, Unit Testing)

Validate the security of the running application and conduct penetration testing.

- Run Dynamic Application Security Testing (DAST) to find live vulnerabilities with **OWASP ZAP**
- Run automated unit tests as part of the pipeline

#### 5. Pre-Production (Kubernetes Security & Policy Enforcement through OPA with Gatekeeper)

Enforce Kubernetes security policies, role-based access control (RBAC), and network security.

- Validate Kubernetes security policies using Open Policy Agent (OPA)/Gatekeeper
- Enforce network security & API rate limits using Istio (mTLS, ingress policies)

#### 6. Production (Monitoring & Logging, RASP, Falco)

In production, shift to continuous monitoring and threat detection.

- Monitor live Kubernetes clusters for security anomalies using **Falco**
- Use Runtime Application Self-Protection (RASP) with **OWASP AppSensor**
- Implement logging, tracing and monitoring with **Grafana + Loki + Prometheus** stack