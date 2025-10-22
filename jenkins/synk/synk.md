# Snyk Notes

## Example DevSecOps Workflow

- Snyk test in local dev and pre-commit.
- Snyk monitor for continuous protection.
- Docker image scanned via snyk container test in CI.
- IaC files tested with snyk iac test before infra changes.
- GitHub/Jenkins integrates snyk test in pipelines.
- Vulnerabilities reported to GitHub Security or Snyk Dashboard.
- Policy enforcement blocks merges based on severity.

## Installation

### Add Snyk to Your Jenkins Pipeline

✅ Step 1: Get a Snyk Token
- Go to https://snyk.io and log in (GitHub, Google, etc.).
- In the bottom-left, click your profile → Account Settings.
- Copy the auth Token.

c1ba0099-d33c-4b6f-bed1-80a446b75e64

To install in Linux

```bash
curl https://static.snyk.io/cli/latest/snyk-linux -o snyk && chmod +x snyk && mv snyk /usr/local/bin/
```

### Pro Tip (from the trenches)

A common best practice is:

- Use Snyk for developer-focused scanning (dependencies, auto-fixes)
- Use Trivy for CI pipeline hardening (containers, IaC, SBOMs)

### Jenkins Environment Configuration

```groovy
environment {
    SCANNER_HOME=tool 'sonar-scanner'
    SNYK_TOKEN = credentials('SNYK_TOKEN')
}
```

## Snyk Commands

### JSON Output
```bash
snyk test > snyk-output.txt
snyk test --json > snyk-report.json
snyk test --sarif > snyk-report.sarif

snyk container test my-app:latest --file=Dockerfile --json > snyk-docker.json

snyk iac test --json > snyk-iac.json
snyk k8s test --file=k8s/deployment.yaml --json > snyk-k8s.json
snyk code test --json > snyk-code.json
```

### Table Output
```bash
# Open Source Dependency Scan (table format)
snyk test > snyk-output.txt

# Container scan (table format)
snyk container test my-app:latest --file=Dockerfile > snyk-container.txt

# IaC scan (table format)
snyk iac test > snyk-iac.txt

# Kubernetes YAML scan (table format)
snyk k8s test --file=k8s/deployment.yaml > snyk-k8s.txt

# Static Code Scan (table format)
snyk code test > snyk-code.txt
```