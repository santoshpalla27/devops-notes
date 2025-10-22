# Trivy

Trivy is used to find vulnerabilities in source code docker-image etc

Trivy finds known vulnerabilities iac issues misconfigurations sensitive and secret information and software license

Trivy is used to scan kubernetes-cluster docker-image github-repo etc

## Installation

To install trivy use below command

```bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo sh -s -- -b /usr/local/bin

trivy --version
```

## Targets Which Trivy Can Scan

- container-image
- filesystem
- git repo
- virtual machine image
- Kubernetes
- aws

## Trivy Commands

```bash
trivy image imagename  # to scan a image

trivy fs path  # to scan a local directory

trivy fs --format table -o fs.html .  # This will scan your project folder. Generate a table-formatted report. Save it to fs.html (though it'll just be plain text with .html extension).

trivy repo repo-url  # to scan GitHub repo

trivy image --format json --output results.json image-name  # to scan and give output in json format

trivy image image-name --severity HIGH,CRITICAL  # to scan for specific severity levels

trivy fs . > results.txt  # this will save the exact table result in the file
```