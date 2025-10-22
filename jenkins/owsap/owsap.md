# OWASP Notes

OWASP is an dependency scanning tool

It analyzes declared dependencies, NOT installed ones.

| Tool | Checks Code Files | Scans Installed Packages |
|------|-------------------|--------------------------|
| OWASP DC | Yes (pom.xml, package.json, etc.) | ❌ No |
| Snyk | ✅ Yes | ✅ Yes (via snyk monitor) |
| Trivy (fs) | ✅ Yes (can scan files) | ✅ (also image binaries) |

## How OWASP Works

OWASP Dependency-Check parses:

- pom.xml for Java/Maven
- build.gradle for Gradle
- package.json for Node.js
- requirements.txt for Python
- composer.lock for PHP
- .nuspec or packages.config for .NET

Then it:

- Maps those to CPEs (Common Platform Enumeration)
- Checks those CPEs against the NVD (National Vulnerability Database)

It does NOT look inside node_modules/, .m2/repository, or venv/ folders.

## In Java

When a source code is build it create a war file

War file consists of both dependency mentioned in the pom.xml file and source code

So OWASP is used to scan the war file

This included after mvn build is done

## In Node.js

When a source code is build a package.json is build

So OWASP will scan the package.json for the vulnerabilities

This is included after npm install

## Installation in Jenkins

Go to plugins and available plugins and search for owasp dependency-check plugin and install

Go to tools find Dependency-Check installations and give a name as owasp (this should match with the code) and install automatically using GitHub.com

API KEY: f4347015-0afd-424c-9043-6ec4ccacfc77

### How to fix it:

1. Get a Free NVD API Key
   Go to: https://nvd.nist.gov/developers/request-an-api-key
   
   Takes 30 seconds to request.
   
   You'll get an API key in your email instantly.

2. Configure OWASP Plugin in Jenkins with the API Key:
   Go to: Jenkins → Manage Jenkins → Global Tool Configuration

Add api-token in credentials as owasp and use the code below

### To scan Java code

```groovy
withCredentials([string(credentialsId: 'owasp', variable: 'NVD_API_KEY')]) {
    dependencyCheck additionalArguments: ['--scan', 'target/', '--nvdApiKey', NVD_API_KEY], odcInstallation: 'owasp'
    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
}
```

### To scan MERN stack code

```groovy
dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'owasp'
dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
```

```groovy
stage('OWASP Dependency-Check Scan') {
            steps {
                dir('mern/frontend') {
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'owasp'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
```