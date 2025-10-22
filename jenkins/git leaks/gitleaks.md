# GitLeaks Notes

## Installation

```bash
sudo yum update -y
sudo yum install -y unzip curl git


curl -s https://api.github.com/repos/gitleaks/gitleaks/releases/latest \
| grep "browser_download_url.*linux_x64" \
| cut -d '"' -f 4 \
| wget -qi -

chmod +x gitleaks_*.tar.gz
tar -xvzf gitleaks_*.tar.gz
sudo mv gitleaks /usr/local/bin/

gitleaks version
```

## To Detect in Linux

```bash
git clone https://github.com/Plazmaz/leaky-repo.git
cd leaky-repo
gitleaks detect
```

```bash
gitleaks detect --source . -r gitleaks-report.csv  # to scan the repo u r in and generate a report in csv
gitleaks detect --source . -r gitleaks-report.json  # to scan the repo u r in and generate a report in json (best)
gitleaks detect --source . --log-opts="-10" -r gitleaks-report.json  # to scan the repo for last 10 commits and generate a report in json
```

## In Jenkins

```groovy
stage('git leaks') {
            steps {
                sh 'gitleaks detect --source . -r gitleaks-report.json -f json'
            }
        }
```

This will check the working repo for the leaks also fails the pipeline when there are any leaks as it returns 1 as exit status

If you want pipeline to not fail even after finding leaks then just add || true

```groovy
stage('git leaks') {
            steps {
                sh 'gitleaks detect --source . -r gitleaks-report.json -f json || true'
            }
        }
```