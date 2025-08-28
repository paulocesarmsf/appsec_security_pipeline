# Application Security Pipeline

### Applications used

- TBD (insert links to the repositories of the applications selected for analysis)

### Tools selected for SAST, DAST, SCA (why chosen + links).
- SAST: [Semgrep](https://semgrep.dev/ "Official Semgrep Website")
- SCA: [OSV-Scanner](https://github.com/google/osv-scanner "Google OSV Scanner GitHub Repository")  
- DAST: [OWASP ZAP](https://www.zaproxy.org/ "OWASP Zed Attack Proxy Official Website")

These tools were selected because they support a broad range of languages, enabling a flexible pipeline that can be applied across different teams and repositories.They also provide official GitHub Actions, which simplifies integration and ongoing maintenance for the AppSec team. Finally, they are well-established and widely recognized within the security community, ensuring reliability and strong community support.

### Steps to Reproduce Security Pipeline (Scans) in CI/CD
This guide shows how to run two security pipelines and how they fit in your CI/CD:

- **Static – Security Pipeline (PRE build & deploy)**  
  Runs SAST (Semgrep) and SCA (OSV-Scanner) against source code **before** you build and deploy.

- **Dynamic – Security Pipeline (POST build & deploy)**  
  Runs DAST (OWASP ZAP full scan) against a **running** environment **after** you deploy.

Both workflows support:
- **Manual runs** via `workflow_dispatch` (GitHub UI/CLI/API)
- **Reusable invocation** via `workflow_call` from another repository

## 1) Run Manually (workflow_dispatch)

### Static — UI
1. Open **Actions** in the repo that hosts `static-security-pipeline.yml`
2. Select **Static – Security Pipeline** → **Run workflow**
3. Fill inputs:
   - `target_repo`: e.g., `https://github.com/example.git`
   - `target_ref` (optional): e.g., `main`
4. Click **Run workflow**

### Dynamic — UI
1. Open **Actions** in the repo that hosts `static-security-pipeline.yml`
2. Select **Dynamic – Security Pipeline** → **Run workflow**
3. Fill inputs:
   - `target_url`: e.g., `https://yourapp.example.com`
4. Click **Run workflow**

## 2) Reuse from Another Repository (workflow_call)

### Static — Caller Workflow Example
```yaml
name: Call – Static Security Pipeline

on:
  workflow_dispatch:

jobs:
  run-security-pipeline:
    uses: OWNER/SECURITY-PIPELINE-REPO/.github/workflows/static-security-pipeline.yml@main
    with:
      target_repo: "https://github.com/flaskbb/flaskbb.git"
      target_ref: "master"
    permissions:
      contents: read
```

### Dynamic — Caller Workflow Example
```yaml
name: Call – Static Security Pipeline

on:
  workflow_dispatch:

jobs:
  run-security-pipeline:
    uses: OWNER/SECURITY-PIPELINE-REPO/.github/workflows/static-security-pipeline.yml@main
    with:
      target_repo: "https://github.com/flaskbb/flaskbb.git"
      target_ref: "master"
    permissions:
      contents: read
```