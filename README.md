# Application Security Pipeline

### Applications used

- TBD (insert links to the repositories of the applications selected for analysis)

### Tools selected for SAST, DAST, SCA (why chosen + links).
- SAST: [Semgrep](https://semgrep.dev/ "Official Semgrep Website")
- SCA: [OSV-Scanner](https://github.com/google/osv-scanner "Google OSV Scanner GitHub Repository")  
- DAST: [OWASP ZAP](https://www.zaproxy.org/ "OWASP Zed Attack Proxy Official Website")

These tools were selected because they support a broad range of languages and frameworks, enabling a flexible pipeline that can be applied across different teams and repositories. 
They also provide official GitHub Actions, which simplifies integration and ongoing maintenance for the AppSec team. Finally, they are well-established and widely recognized within the security community, ensuring reliability and strong community support.

### High-level CI/CD integration

The pipeline was designed to be dynamic and generic, so it can be reused in different scenarios:

- Execution during SDLC (post-commit (Static) or build/deploy (Dynamic))
    - Workflows can be automatically triggered on push, pull request, or after a deploy event.

- On-demand execution by the AppSec team
    - The pipeline can also be triggered manually for ad-hoc assessments of specific applications.

- Infrastructure
    - Built entirely with GitHub Actions.
    - Results are exported both to GitHub Code Scanning Alerts (SARIF) and artifacts that can be downloaded by the team.

### Threat Modeling
- TBD