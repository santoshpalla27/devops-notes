# SonarQube Notes

```bash
-Dsonar.exclusions=**/test/**,**/*.spec.js  # to make SonarQube not scan the given files
-Dsonar.exclusions=fs.html,image.html,dependency-check-report.xml  # to skip sec reports
```

For SonarQube to scan code you need to install the dependency to like if your code is designed in javascript you need to install nodejs in you ec2 server
or as an Jenkins plugin or in the ec2-server

We install SonarQube scanner tool to integrate ec2 machine SonarQube with Jenkins

SonarScanner for MSBuild installations -- used for .net applications

## Installation

First install docker
Install Jenkins
Also install git

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade
sudo yum install  java-17-amazon-corretto-devel -y
sudo yum install jenkins -y
systemctl start jenkins
systemctl enable Jenkins

yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker

yum install git -y
```

## Running SonarQube

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:latest or  sonarqube:lts-community
```

You should both Jenkins 8080 and sonarqube 9000 port in the security group

And login of sonarqube is admin and admin as username and password

## SonarQube Without SonarQube Scanner

Go to right side icon and my account

And generate a sonar token for auth and type user token and no expiration and generate and copy very important once missed cannot be viewed again

```bash
mvn sonar:sonar \
  -Dsonar.host.url=http://52.72.246.253:9000/ \
  -Dsonar.login=squ_6f56e281fe762dd139e643fb81a2a9b21fa7183f
```

```groovy
pipeline {
    agent any

    stages {
        stage('poll scm') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/santoshpalla27/sample-java.git'
            }
        }
    
        stage('build') {
            steps {
                sh 'mvn clean install'
            }
        }
    
    
        stage('sonar analysis') {
            steps {
                sh ''' mvn sonar:sonar \
                      -Dsonar.host.url=http://52.72.246.253:9000/ \
                      -Dsonar.login=squ_6f56e281fe762dd139e643fb81a2a9b21fa7183f '''
            }
        }
    }
}
```

## SonarQube Overview

SonarQube is an open-source platform used for continuous code quality inspection and code security. It integrates with your CI/CD pipeline and helps identify bugs, vulnerabilities, code smells, and other potential issues in your source code. It's widely used in software development and DevOps practices to maintain high standards of code quality and security.

SonarQube covers:

- Code quality check --> bugs,code smell,vulnerabilities
- Code coverage -->  amount of code covered in test cases if your code coverage has above 80% then it is considered as good code

Quality profiles are collections of rules to apply during an analysis. These are the rules applied on projects. These are rules that are used to find bugs and vulnerabilities in the project

To add more rules to the quality profile we need to copy the new the existing profile with new name and you can view the new quality profile in quality-profile section

Now to activate more rules you need to go the quality-profile and click activate more and click activate select the level

You can add new quality-profile in administration and marketplace

## Installing SonarQube Scanner in Jenkins

Now to install sonarqube-scanner in Jenkins go to plugin and search sonarqube-scanner and install

Go to manage-jenkins->tools to install the SonarQube scanner tool click on add SonarQube scanner
Give a name like sonar-scanner and remember it and select the version and select install automatically

To generate sonar-token
Go to administration->security->users and under tokens section click fourlines icon
Give a name and expiry and generate a token and record it somewhere cannot be viewed again  squ_c5e596b4572e5743175868241ecd4d3bbb4b53ce

Now go to system and configure sonar-server info enable the environment variable option and add SonarQube
Give a name like sonar-server and server url will be the SonarQube url and server auth token will be sonarqube-token and has to be added as secret text
And save

```groovy
pipeline {
    agent any
    tools {
        nodejs 'nodejs'  // this should match the tools installation configuration name like nodejs is default for nodejs tool and 'nodejs' is name that should match
    }
    environment  {
        SCANNER_HOME=tool 'sonar-scanner'  // this should match the sonar-scanner name configured in tools
    }   
    stages {
        stage('poll scm') {
            steps {
                git branch: 'main', url: 'https://github.com/santoshpalla27/mern-todo-app.git'
            }
        }
        stage('sonar scan') {
            steps {
                withSonarQubeEnv('sonar-server') {   // sonar-server should match with the name configure in the system configuration
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=three-tier-frontend \
                        -Dsonar.projectKey=three-tier-frontend \
                        -Dsonar.sources=. '''   // this will scan all the directories including the sub directories
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    def qualityGate = waitForQualityGate()  // waits for the quality gate result
                    if (qualityGate.status != 'OK') {  // if the status is not OK, it will fail the build
                        error "Quality Gate failed: ${qualityGate.status}"
                    }
                    else{
                        echo "Quality Gate passed"
                    }
                }
            }
        }
    }
}
```

To make SonarQube scan specific directory

```groovy
stage('sonar scan') {
            steps {
                dir('mern/frontend'){
                    withSonarQubeEnv('sonar-server') {   // sonar-server should match with the name configure in the system configuration
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=frontend \
                        -Dsonar.projectKey=frontend \
                        -Dsonar.sources=.
                        '''
                    }
                }    
            }
        }
```

## Creating a Project in SonarQube

To be more precise if you want to create your own project manually go to projects and create project give name and key and set up and and choose locally choose create or give existing token and choose the language of the project
And change the code accordingly

```groovy
withSonarQubeEnv('sonar-server') {   // sonar-server should match with the name configure in the system configuration
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=frontend \
                        -Dsonar.projectKey=frontend '''
```

## SonarQube Quality Gate

First we need to create a webhook so that Jenkins pipeline receives the replay with quality-gate info
To create go to administration->configurations and webhooks and create
Give a name and in url http://jenkins-url:8080/sonarqube-webhook/ and create

```groovy
stage('Quality Gate') {
            steps {
                script {
                    def qualityGate = waitForQualityGate()  // waits for the quality gate result
                    if (qualityGate.status != 'OK') {  // if the status is not OK, it will fail the build
                        error "Quality Gate failed: ${qualityGate.status}"
                    }
                }
            }
        }
```

With timeout if the quality-gate doesn't respond in the given time will throw an error

```groovy
stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'MINUTES') {  // waits for a maximum of 1 minutes
                        def qualityGate = waitForQualityGate()  // waits for the quality gate result
                        if (qualityGate.status != 'OK') {  // if the status is not OK, it will fail the build
                            error "Quality Gate failed: ${qualityGate.status}"
                        } else {
                            echo "Quality Gate passed"
                        }
                    }
                }
            }
        }
```

## SonarQube and PostgreSQL for Data Persistence

```yaml
version: '3.8'

services:
  postgresql:
    image: postgres:15
    container_name: sonarqube_db
    restart: unless-stopped
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonarqube
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - sonarnet

  sonarqube:
    image: sonarqube:latest
    container_name: sonarqube
    restart: unless-stopped
    depends_on:
      - postgresql
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://postgresql:5432/sonarqube
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    ports:
      - "9000:9000"
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    networks:
      - sonarnet

volumes:
  postgres_data:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:

networks:
  sonarnet:
```

To increase virtual memory for elastic search

```bash
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

The root folder for volumes is /var/lib/docker/volumes

```bash
tar -czvf my_archive.tar.gz my_directory/ dir/  # to zip all the dir

tar -xzvf my_archive.tar.gz -C /path/to/destination/  # to unzip
```

## For Java

```groovy
pipeline {
    agent any
    tools{
        jdk 'java'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/santoshpalla27/santosh-online-botique.git'
            }
        }
        stage('version') {
            steps {
                sh 'java -version'
            }
        }
        stage('sonar scan') {
            steps {
                dir('src/adservice'){
                    withSonarQubeEnv('sonar-server') {   // sonar-server should match with the name configure in the system configuration
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=adservice \
                        -Dsonar.projectKey=adservice \
                        -Dsonar.java.binaries=. \
                        -Dsonar.sources=.
                        '''
                    }
                }    
            }
        }
    }    
}
```

```bash
-Dsonar.java.binaries=. \         # -- include this
```