# Application Security Pipeline

## Applications used

- TBD (insert links to the repositories of the applications selected for analysis)

## High-level CI/CD integration diagram and explanation.
This security pipeline was built on **GitHub Actions** to be flexible and easy to integrate into different repositories. It can be triggered automatically within development workflows or executed manually by the security team.

The pipeline is divided into two main workflows:

1. **Static Security Pipeline (Pre-build & Pre-deploy)**
    - SAST (Static Application Security Testing): Runs [Semgrep](https://semgrep.dev/ "Official Semgrep Website")
    - SCA (Software Composition Analysis): Runs [OSV-Scanner](https://github.com/google/osv-scanner "Google OSV Scanner GitHub")

2. **Dynamic Security Pipeline (Post-build & Post-deploy)**
    - DAST (Dynamic Application Security Testing): Runs [OWASP ZAP](https://www.zaproxy.org/ "OWASP Zed Attack Proxy Official Website")

These tools were selected because they support a broad range of languages, enabling a flexible pipeline that can be applied across different teams and repositories.They also provide official GitHub Actions, which simplifies integration and ongoing maintenance for the AppSec team. Finally, they are well-established and widely recognized within the security community, ensuring reliability and strong community support.

![alt text](/CICD/static/workflows.png)

## Steps to Reproduce Security Pipeline (Scans) in CI/CD
This guide shows how to run two security pipelines and how they fit in your CI/CD:

Both workflows support:
1. **Manual runs** via `workflow_dispatch` (GitHub UI/CLI/API)
2. **Reusable invocation** via `workflow_call` from another repository

## 1) Run Manually (workflow_dispatch)

### Static — UI
1. Open **Actions** in this repository.
2. Select **Static – Security Pipeline** → **Run workflow**
3. Fill inputs:
   - `target_repo`: e.g., `https://github.com/example.git`
   - `target_ref` (optional): e.g., `main`
4. Click **Run workflow**

![alt text](/CICD/static/image.png)

### Dynamic — UI
1. Open **Actions** in this repository.
2. Select **Dynamic – Security Pipeline** → **Run workflow**
3. Fill inputs:
   - `target_url`: e.g., `https://yourapp.example.com`
4. Click **Run workflow**

![alt text](/CICD/static/image-1.png)

## 2) Reuse from Another Repository (workflow_call)

### Static — Caller Workflow Example
```yaml
name: CI – Call Static Security Pipeline (Fixed target)

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  run-security-pipeline:
    uses: paulocesarmsf/appsec_security_pipeline/.github/workflows/static-security-pipeline.yml@main
    with:
      target_repo: ${{ format('https://github.com/{0}.git', github.repository) }}
      target_ref:  ${{ github.event_name == 'pull_request' && github.head_ref || github.ref_name }}
    permissions:
      contents: read
```

### Dynamic — Caller Workflow Example
```yaml
name: CI – Call Dynamic Security Pipeline (Vars)

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  run-dast:
    uses: paulocesarmsf/appsec_security_pipeline/.github/workflows/dynamic-security-pipeline.yml@main
    with:
      target_url: ${{ vars.TARGET_URL }}
    permissions:
      contents: read
```