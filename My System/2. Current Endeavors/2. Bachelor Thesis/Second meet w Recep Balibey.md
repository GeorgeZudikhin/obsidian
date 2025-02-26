### 1️⃣ AWS EKS, Azure AKS, and GCP GKE: Will you be running tests on all three, or will you focus more on one provider for in-depth analysis?

- The primary in-depth testing will be conducted on **AWS EKS**, as it is the primary deployment target for the project.
- Comparative evaluation will be performed on **Azure AKS and GCP GKE**, focusing on security features, compliance readiness, and ease of policy enforcement rather than running the full pipeline on each provider.
- The comparison will analyze:
    1. Security Tools & Features (AWS GuardDuty vs. Azure Defender vs. GCP Security Command Center).
    2. Built-in Compliance Support (Which provider makes PCI DSS, SOC 2, and ISO 27001 compliance easier?).
    3. Ease of Policy Enforcement (How well do OPA, Kyverno, and Istio integrate on each platform?).


---

### 2️⃣ Implementation & Technical Feasibility

#### **What specific attack scenarios will you use for testing security effectiveness in each pipeline stage?**

Each stage of the **DevSecOps pipeline** will be tested with **controlled attack scenarios**, simulating real-world threats.

| **Pipeline Stage**                          | **Attack Scenario**                                           | **Test Method**                                    | **Expected Outcome**                         |
| ------------------------------------------- | ------------------------------------------------------------- | -------------------------------------------------- | -------------------------------------------- |
| Develop                                     | Hardcoded API keys in source code                             | Commit API key & test detection by pre-commit hook | Security hook should block the commit        |
| Build (SAST & SCA)                          | Inclusion of an outdated, vulnerable dependency (e.g., Log4J) | Push vulnerable code and observe detection         | SAST/SCA tools should flag the issue         |
| Acceptance Testing (DAST & Pentesting)      | SQL Injection on API                                          | Run OWASP ZAP against staging API                  | Vulnerability should be detected and blocked |
| Pre-Production (OPA, Kyverno, Istio)        | Deploy a privileged container                                 | Apply security policies and attempt to deploy      | Deployment should be blocked                 |
| Production (Runtime Security, Falco, Istio) | Unauthorized lateral movement attempt between microservices   | Simulate unauthorized API call                     | Istio should detect & block traffic          |

Each stage will have targeted attack simulations that measure detection rates, false positives, and prevention effectiveness.

---

#### **Will you automate the process of injecting vulnerabilities for evaluation? If so, how?**

Yes, vulnerability injection will be automated to ensure consistency and reproducibility.

Automation Methods:

1. **For Code-Level Vulnerabilities (SAST & SCA)**
    
    - Use a **pre-defined repository** with known vulnerabilities (intentionally vulnerable microservices).
    - Implement a CI/CD step that automatically modifies code to introduce vulnerabilities, testing tool detection rates.
2. **For Runtime Vulnerabilities (DAST, Istio, Falco)**
    
    - Use **automated attack scripts** to attempt **SQL injection, XSS, and privilege escalation** in a controlled test environment.
    - Deploy misconfigured Kubernetes manifests automatically (e.g., privileged containers, open network policies) to test OPA/Kyverno effectiveness.
3. **For Network & API Attacks (Istio, Egress Control, Lateral Movement Prevention)**
    
    - Use **locust** or **k6** for automated **DDoS and rate-limiting testing**.
    - Deploy unauthorized API calls via automated scripts to verify Istio’s policy enforcement.

✅ Automated scripts will be used for security misconfigurations, API abuse, and runtime threats to evaluate effectiveness **at scale**.

---

### 3️⃣ Evaluation & Metrics: How will you define the acceptable threshold for security overhead in pipeline execution?

To determine the **acceptable security overhead**, we will compare:

- Pipeline execution time with and without security enforcement.
- CPU/memory overhead caused by security tools (SAST, DAST, OPA, Istio, Falco, etc.).

✅ **Defining the Threshold:**

1. **Pipeline Delay:**
    
    - Acceptable overhead: **≤ 30% increase** in execution time.
    - Example: If the build process without security takes 5 min, a final build under 6m 30s is acceptable.
2. **Security vs. Detection Rate Trade-off:**
    
    - If adding a security tool increases pipeline time by 20% but improves vulnerability detection by 50%, it is considered acceptable.
3. **Resource Utilization (CPU/Memory Overhead):**
    
    - Security tools should **not exceed 15% CPU/memory overhead** on average.
    - Example: If a standard pod consumes **500MB RAM, security monitoring should not exceed an additional 75MB RAM**.

Acceptable thresholds will be defined based on a **30% maximum pipeline overhead and 15% max CPU/memory overhead**, balancing security and efficiency.

---

### **4️⃣ Timeline & Next Steps: Have you identified any risks that might delay the project, such as tool compatibility or cloud resource limitations?**

Yes, the following risks have been identified, along with mitigation strategies:

| **Potential Risk**                | **Description**                                                                                            | **Mitigation Strategy**                                                                                                   |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Tool Compatibility Issues         | Some security tools (e.g., Istio, Falco, Kyverno) might not work seamlessly across all cloud providers.    | Focus main testing on **AWS EKS** and ensure compatibility before testing Azure AKS/GCP GKE.                              |
| Cloud Resource Limitations        | Running full security scans, Istio traffic rules, and penetration tests can consume significant resources. | Use **auto-scaling policies** and **scheduled security tests** instead of running full tests on every pipeline execution. |
| False Positives in Security Tools | SAST/DAST tools might produce high false positives, affecting analysis.                                    | Implement **manual verification** for flagged issues and compare overlapping results between tools.                       |
| Time Constraints                  | Full end-to-end security evaluation may take longer than expected.                                         | Prioritize **key security controls first**, then expand based on available time.                                          |

Risks have been identified, and mitigation strategies will ensure that **tool compatibility, cloud resource usage, and time constraints do not delay the research**.

---

### **Final Takeaways:**

✅ **AWS EKS will be the primary test environment, with security feature comparisons for Azure AKS & GCP GKE**.  
✅ **Specific attack scenarios** will be tested for **each pipeline stage**, targeting real-world vulnerabilities.  
✅ **Automated vulnerability injection & runtime testing** will be used for **consistency & scalability**.  
✅ **Acceptable security overhead** is defined as **≤30% pipeline delay, ≤15% CPU/memory usage increase**.  
✅ **Risks such as tool compatibility, cloud limitations, and false positives** have been identified & mitigated.


Each pipeline stage → specific attacks
Inject automatically? metasploit, owasp zap
acceptable threshold?
