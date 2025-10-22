# Jenkins Notes

## Jenkins Management

Manage Jenkins moved to right top as setting button

## Stash and Unstash

Stash and unstash steps in Jenkins Pipeline are used to share files between stages, especially when those stages run on different agents or workspaces.
They allow you to preserve and transfer workspace contents (e.g., build artifacts, source code, configuration files) from one stage to another.

Jenkins downloads source code only in the workspace where checkout scm is called.
If later stages:
- Use the same workspace and agent → ✅ source code persists.
- Use a different agent (Docker/container/another node) → ❌ the workspace is new, so Git repo isn't present unless you do another checkout.

## Global Tool Configuration

In short, tools which are installed automatically are used in tools sections mostly build tools and cli tools are installed in vm

Global Tool Configuration in Jenkins

In Jenkins, there's a section called Manage Jenkins → Global Tool Configuration.
Here, you can define "tools" like Maven, JDK, Ansible, Gradle.

For each tool, you can give:
- Name → how you refer to it in your pipeline (tools { maven 'maven' }).
- Install automatically → Jenkins will download and install the tool on the agent when the pipeline runs.

### What "skip manual install" means

Normally, if you wanted to run Maven on an agent, you'd have to log into the agent and apt install maven or yum install maven.
With Jenkins Global Tool Configuration + Install Automatically, you don't need to do that manually.
Jenkins will download Maven (or Ansible, etc.) at runtime on the agent.
The pipeline can just reference it by name in the tools block.

```
Pipeline Step (needs Maven)
        |
        v
tools { maven 'maven' } → Jenkins checks agent
        |
        |-- Maven exists? → use it
        |-- Maven missing? → Jenkins downloads & installs automatically
        |
        v
Pipeline runs Maven command
```

Only tools that Jenkins knows how to install via Global Tool Configuration support this feature.
CLI tools like Docker, kubectl, Git are not supported here; their plugins only integrate Jenkins with the tool, they don't install the binary.

### What is the tools {} section?

The tools {} block in a Jenkins Pipeline (Declarative Pipeline) allows you to automatically install and use build tools pre-configured in:

Jenkins → Global Tool Configuration

This is useful for consistency across builds. However, not all tools support auto-installation.

### Recommendation

Keep in Jenkins:
- Build-specific tools like Maven, Java, Gradle, Ant
- Sonar plugins (through Jenkins GUI)

Keep on EC2 (via yum/dnf or apt):
- SCM tools (Git)
- Infra/DevOps tools (Ansible, Docker, kubectl, terraform)
- Other system dependencies (ffmpeg, unzip, zip, etc.)

### Pro Tip: Use Docker Agent for Tool Isolation

If you're working in CI/CD workflow and want to avoid installing tools manually all the time, use Docker agents with pre-baked build environments.

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.9.3-eclipse-temurin-21'
        }
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

## Docker Agent

### What is a Docker Agent in Jenkins?

A Docker agent tells Jenkins to run a particular pipeline (or stage) inside a Docker container, instead of running it directly on the EC2/VM.

It's like saying:

"⚙️ Run this build inside an isolated, temporary Docker container with the tools I need."

### How to Set Up Docker Agent in Jenkins

✅ Prerequisites

Docker must be installed and running on the Jenkins agent or controller.

✅ Step-by-Step Setup

🔹 Step 1: Install Docker in EC2 (Jenkins running on EC2)

```bash
# Amazon Linux 2023 / 2
sudo yum install docker -y              # or `dnf` in Amazon Linux 2023
sudo systemctl start docker
sudo systemctl enable docker

# Give Jenkins permission to use Docker
sudo usermod -aG docker jenkins

# Restart the instance or Jenkins
```

🔹 Step 2: Install Jenkins Docker Plugins
In Manage Jenkins → Plugins, install:

- Docker Pipeline ✅
- Docker Commons Plugin ✅
- Optionally: Docker Plugin (for remote Docker server connectivity, not always needed)

Step 3: Add Docker Host Configuration (Optional)
For isolated Docker nodes, go to:

Manage Jenkins → Manage Nodes and Clouds → Configure Clouds → Add Docker

…but this is needed only for remote Docker agents. You can skip this if you're just using Docker from the local Jenkins agent.

### Can You Use Jenkins Plugins/Tools Inside Docker Agent?

❌ No, not directly.
Tools (JDK, Maven, Gradle, etc.) configured in:

Manage Jenkins → Global Tool Configuration

…are installed on the host OS (your EC2 VM), not inside your Docker containers.

TL;DR First

| Feature / Plugin                          | Available in Docker Agent? | How to Use |
|------------------------------------------|---------------------------|------------|
| Jenkins Plugins (like SonarQube Scanner) | ✅ Yes — on Jenkins side   | Controlled via Jenkins, not inside the container |
| Sonar Scanner CLI                        | ❌ No — unless you install it in the container | Use plugin in Jenkins OR install in Docker image |
| kubectl (Kubernetes CLI)                 | ❌ No — unless in the Docker image | Add to Docker image manually |
| docker login (CLI)                       | ❌ No — unless Docker CLI is in the container AND Docker is accessible | Use host Docker or run Docker-in-Docker |
| Credentials (e.g. for Docker or Sonar)   |                           |            |

To use different java version for different pipeline just add more jdk in tools section and match the name in pipeline

## Commands and Configuration

```bash
mount -o remount,size=2G /tmp/  # to remount tmp with more the storage size

which ansible  # to find the ansible bin path

yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
usermod -aG docker jenkins
systemctl restart docker
systemctl restart jenkins  # to run docker on jenkins
```

4 space syntax

```groovy
environment  {
    SCANNER_HOME=tool 'sonar-scanner'  // this should match the sonar-scanner name configured in tools
}   
```

## Docker Compose

```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

Need to create a user named Jenkins and add user to sudoers group and login as Jenkins user

```bash
sudo usermod -aG docker $USER  # to run docker on user Jenkins
newgrp docker  # just run
```

## Docker Compose Commands

```bash
docker-compose up -d  # to up the docker-compose
docker-compose down   # to down and delete the docker container 
docker-compose build  # to build the image in the docker-compose file
```

The three line name in the tools should be the first and the label will be second no matter capital letter or small ex

```
jdk "java"
maven "maven"
```

Tools name for maven maven integration tool
For java jdk installation Eclipse Temurin installer

## Jenkins Tools

- Eclipse Temurin installer -- to install java jdk 
- owasp dependency-check plugin -- owsap dependency check tool
- SonarQube Scanner -- to install SonarQube scanner
- backup ,thinbackup,perodic backup --- to do backup of jenkins
- pipeline stage view and blue ocean -- to view the pipeline in better way
- maven integration -- to integrate maven
- Pipeline Maven Integration Plugin -- to integrate maven
- SSH -- to make sh communication between servers 
- deploy to container -- to deploy war apps to container 
- Docker pipeline  , Docker -- for docker commands
- Kubernetes and Kubernetes cli -- Kubernetes
- ansible plugin -- for ansible
- Config File Provider Plugin -- used to edit config files

While defining tools

```groovy
tools {
    maven 'maven'      // 'maven' is the name you gave in Global Tool Configuration
    jdk 'jdk'          // 'jdk' is the name you gave in Global Tool Configuration
    ansible 'ansible'  // 'ansible' is the name you gave in Global Tool Configuration
}
```

## Installing Jenkins in Linux

```bash
# Add Jenkins repo
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Upgrade packages
sudo dnf upgrade -y

# Install Java 21 (compatible with Jenkins 2.426+)
sudo dnf install java-21-amazon-corretto-devel -y

# Install Jenkins
sudo dnf install jenkins -y

# Start and enable Jenkins service
sudo systemctl daemon-reexec
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```

Enable port 8080 in sg

Failed to connect to repository : Error performing git command: git ls-remote -h https://github.com/santosh2708/toy-bootsrap.git HEAD

If you see this error try installing git in Jenkins server

## Shell Script with Parameters

```bash
#!/bin/bash

USERNAME=$1
PASSWD=$2

echo "hello $USERNAME $PASSWD"
```

In Jenkins you to choose this project is parametrized in the setting and add the parameters

In shell to execute you have to give the path to the script and parameters

## Scheduling Jobs in Jenkins

To schedule a job in Jenkins you need to choose

Build triggers and build periodically this will take cron timing as an input and execute job periodically 

```
* * * * * min hr date month day
```

## Auto GitHub Repo Pulling

You have to give the GitHub link in the git section 

In build triggers you have to choose poll scm and it will take cron timing and it will check periodically and if any changes are made it will pull the trigger 

## Discard Old Builds

Discarding old builds helps keep your Jenkins environment clean and prevents the server from getting filled with old logs, artifacts, and build data.

Can be enabled in general setting and give max build and build keep days 

## Connecting SSH in Jenkins

Now to connect ssh in Jenkins you need to install Jenkins plugin called ssh 

Now go to credentials and system global, global credentials and choose add credentials and choose username with pass and in username enter the username of the machine like the user name and copy private key in privatekey enter directly and create

Now to add ssh to the machine go to manage Jenkins and system and search for ssh sites add 
Enter hostname the ip of machine or the name and port 22 credentials chose what we define before and check connection and save

## Environment Variables

Environment variables can be called from any job 
You can declare environment variable from system and global properties and environment variable and add 

Can be used in any job ex echo "$environment" 

## Jenkins with Ansible

First we create 3 container for ansible in first we install ansible and ssh-keygen and copy the pub key to 2 containers

Then we create a new containers for Jenkins and ssh-keygen in it and copy the pub key to main ansible node authorized_keys

Then we go to credentials and create a credential for SSH Username with private key and copy the private key of Jenkins and username of ansible node

Now go to manage jenkins and install ssh plugin and go to system and ssh site and copy the hostname id and port 22 and select username and key in credentials 

Now create a job and choose build steps and ssh with remote host and copy this code cd /etc/ansible ansible-playbook httpd.yaml

## CI/CD of Maven Java

Go to simple-java-maven-app website clone that web using Jenkins git 

Jenkins will clone the repo : /workspace/job-name/rep

Install maven integration plugin (manage Jenkins --> manage Jenkins --> available plugins -> search for maven and install maven integration plugin)

Configure the maven:
manage Jenkins --> tools -> maven installations -> add maven 

Configure maven job --> build steps --> invoke top-level maven targets --> 

Choose maven version,goals seach in Jenkins file in repo

To deploy 

```bash
java -jar path
```

## Connecting SSH in Jenkins (Corrected)

Now to connect ssh in Jenkins you need to install Jenkins plugin called ssh 

Now go to credentials and system global, global credentials and choose add credentials and choose ssh username with private key and in username enter the username of the machine like the user name and copy private key in private key enter directly and create

Now to add ssh to the machine go to manage Jenkins and system and search for ssh sites add 
Enter hostname the ip of machine or the name and port 22 credentials chose what we define before and check connection and save

## Creating Users in Jenkins

Go to manage Jenkins in security select users

Select create user give the credentials and the user will be created

To edit the security for the users created go to manage Jenkins and security and edit the setting

You can change the user permission in Authorization

## Role Based Authorization

To create role based authorization install Role-based Authorization Strategy

Now go to security in authorization select role based authorization and save then go to manage Jenkins in security you will find manage and assign roles

## Safe Restart Jenkins

To safe restart Jenkins 

You need to enter safeRestart to the end of the url like this http://34.228.195.62:8080/manage/safeRestart

And enter you will get a page and enter display banner info and restart

## Build Triggers

### Trigger Builds Remotely

The build can be triggered remotely using script or url etc

Go to triggers and click on trigger build remotely and give a token name and give same in below also edit jobname and url

```
JENKINS_URL/job/testjob/build?token=TOKEN_NAME
```

Edit this and directly paste in browser or give this in scripts

### Build After Other Projects Are Built

This build only after the watch projects match the requirement .. you can go to post build actions and build other projects and give the project name to build with requirements

### Poll SCM

Build the job when there is a change in GitHub and you need to give a cron job to check the GitHub

## Updating Jenkins URL

To update Jenkins url 

Go to manage Jenkins -> system and check for Jenkins location

## Updating Jenkins

To know about system info go to manage Jenkins -> System Information and first view the executable war 

First systemctl stop Jenkins

Go to /usr/share/java and take backup mv jenkins.war bk_jenkins_todaydate.war

Go to download Jenkins war https://www.jenkins.io/download/

And copy the link address of the new version

And now wget https://get.jenkins.io/war/2.488/jenkins.war

systemctl restart Jenkins

systemctl status Jenkins

## My Views in Jenkins

This is used to create separate section of jobs each section can have specified list of jobs they can be created by 

Go to my views in dashboard and click the + symbol give a name and select list view and select the jobs and setting and click ok

## Backup Jenkins in Linux

```
/var/lib/jenkins is the root dir of Jenkins
```

```bash
systemctl stop jenkins

# need to take the backup of jenkins to jenkins_backup and can be saved anywhere 

# to restore first stop Jenkins using systemctl stop Jenkins and get the backup into the /var/lib/Jenkins

# and rm -rf the original Jenkins folder and mv jenkins_backup jenkins

# and most important change ownership of jenkins and all the sub dir

# using chown -R jenkins:jenkins jenkins -- -R for all sub dir 

# now systemctl start jenkins  
```

## Jenkins Master Slave Concept

This is how it is defined

```groovy
pipeline {
    agent { label 'slave1' }
```

Create an ec2 instance 

In main instance go to manage Jenkins->nodes->click new node 

And give a name and select permanent agent and create 

In node setting give the number of executors you want to use and give /tmp as remote root directory cause of no permission issue and give a label (this is used in Jenkins code) and launch method use launch agent via ssh

Now hostname -i copy paste and in credentials set user name as ec2-user and private key copy paste the key which is downloaded and choose manually trusted key verification strategy

Don't select Require manual verification of initial connection

Install java in the slave machine 

Also if needed to create space in tmp use command

```bash
mount -o remount,size=2G /tmp/  # to remount tmp with more the storage size
```

## Installing Custom Java in Linux

To install go to main java download page and find the version you want to install 

Go to /opt and mkdir java8 
And cd to install and wget the new java link
And tar -xvf file to unzip 
And go into the file and pwd and 

Pwd from the main folder just after cd with be java_home and bin pwd will be patch same for windows and linux

```bash
vim /etc/profile.d/java.sh 
```

```bash
cd /opt
wget https://download.java.net/java/GA/jdk11/openjdk-11_linux-x64_bin.tar.gz

#!/bin/bash 
export JAVA_HOME= pwd path paste the pwd of java folder after extraction 
export PATH=${JAVA_HOME}/bin:${PATH}
```

To revert the java version 

```bash
sudo alternatives --config java
```

```
There are 2 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
   1           /usr/lib/jvm/java-17-amazon-corretto.x86_64/bin/java
*+ 2           /usr/lib/jvm/java-23-amazon-corretto.x86_64/bin/java

Enter to keep the current selection[+], or type selection number: ^C
[root@ip-172-31-23-44 opt]# export JAVA_HOME=/usr/lib/jvm/java-23-amazon-corretto.x86_64
export PATH=$JAVA_HOME/bin:$PATH
[root@ip-172-31-23-44 opt]# java -version
openjdk version "23.0.1" 2024-10-15
OpenJDK Runtime Environment Corretto-23.0.1.8.1 (build 23.0.1+8-FR)
OpenJDK 64-Bit Server VM Corretto-23.0.1.8.1 (build 23.0.1+8-FR, mixed mode, sharing)
```

## GitHub Webhook Triggers

Select web hook in Jenkins 
Now go to GitHub repo and in setting you will find webhooks and add webhook
In payload url give Jenkins url/github-webhook/
Content type -> application/json

And rest leave as default and add webhook

## Using Docker Commands in Jenkins

Install Docker pipeline  , Docker plugins

Go to pipeline syntax and use withDockerRegistry: sets up docker registry endpoint

And add docker creds as username and password use docker-username and pass 

In docker installation choose docker

## Change Directory in Jenkins using Dir Command

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

## Remove Docker Images

```groovy
stage('checkout'){
            steps{
            sh 'docker rmi $(docker images -q) || true '
            }
        }
```

Another way

```groovy
script {
    // Get all Docker image IDs
    def images = sh(script: "docker images -q", returnStdout: true).trim()

    // Remove all images if any are found
    if (images) {
        sh "echo '${images}' | tr '\\n' ' ' | xargs -r docker rmi -f"
    } else {
        echo "No Docker images to remove."
    }
}
```

## Binding Credentials to Variables

To bind credentials to variable we use pipeline-syntax and withCredentials()

```groovy
stage('docker push') {
            steps {
                withCredentials([string(credentialsId: 'dockerpass', variable: 'dockerpass')]) {
                    sh "docker login -u santoshpalla27 -p ${dockerpass}"
                }
                sh "docker push ${IMAGE_NAME}"
            }
        }
```

Choose withCredentials:bind credentials to variables

And choose var name and credentials you want to bind

```groovy
withCredentials([string(credentialsId: 'hwllo', variable: 'HELLO')]) {
    // some block
}
```

To bind two or more cred to var

```groovy
pipeline{
    agent any
    stages{
        stage('checkout') {
            steps {
                withCredentials([
                    string(credentialsId: 'hwllo', variable: 'HELLO'),
                    string(credentialsId: 'name', variable: 'NAME')
                ]) {
                    sh 'echo "Hello: $HELLO"'
                    sh 'echo "World: $NAME"'
                }
            }
        }
    }
}
```

## Set AWS Credentials

```groovy
pipeline {
    agent any
    
    parameters {
        choice(name: 'ACTION', choices: ['plan', 'apply', 'destroy'], description: 'Choose Terraform action')
    }
    
    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        TF_WORKING_DIR = 'terraform'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/santoshpalla27/4-tier-project.git'
            }
        }

        stage('Setup AWS Credentials') {
            steps{
                sh '''
                aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                aws configure set region $AWS_REGION
                
                # Verify configuration
                aws sts get-caller-identity
                '''
            }
        }

        stage('Setup Terraform') {
            steps {
                dir(env.TF_WORKING_DIR) {
                    sh 'terraform init'
                }
            }
        }

        stage('Plan Terraform') {
            steps {
                dir(env.TF_WORKING_DIR) {
                    sh 'terraform plan -out=tfplan'
                }
            }
        }

        stage('Apply or Destroy Terraform') {
            when {
                expression { params.ACTION == 'apply' || params.ACTION == 'destroy' }
            }
            steps {
                dir(env.TF_WORKING_DIR) {
                    script {
                        if (params.ACTION == 'apply') {
                            sh 'terraform apply -auto-approve tfplan'
                        } else if (params.ACTION == 'destroy') {
                            sh 'terraform destroy -auto-approve'
                        }
                    }
                }
            }
        }
    }
}
```

## Integrate Ansible with Jenkins

First install ansible plugin 

Go to manage Jenkins > tools 

Give ansible as name 

In path give bin path of ansible = "/usr/bin"  or install automatically

Now go to pipeline syntax and use ansible playbook 

Give path in workspace of ansible playbook and hosts 

And ssh credentials of host machine

Scroll down and check disable the host ssh key check

And leave everything and copy the paste the generated output

This SSH key (likely a private key) allows Jenkins (via Ansible) to connect to your remote host (EC2/VM/etc.) over SSH to run commands.
This must match a public key already configured in /home/ec2-user/.ssh/authorized_keys (or similar) on the target server.
credentialsId: 'ssh' uses this key to authenticate.

=================

```groovy
pipeline {
    agent any
    tools {
        ansible 'ansible'   // This is the name you gave in Global Tool Configuration
    }
    stages {
        stage('Checkout Code') {
            steps {
                ansiblePlaybook credentialsId: 'ssh', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: '/etc/ansible/playbook.yml', vaultTmpPath: ''
                }
            }
        }
    }
}
```

```yaml
- name: deploy to eks cluster
  hosts: appserver 
  gather_facts: no
  become: yes
  tasks: 
    - name: copy deployment to eks cluster
      copy: 
        src: /var/lib/Jenkins/cicd/Ansible/k8s_deployments.yaml
        dest: /home/ec2-user/
```

==========

Recommendation: Organize It Like This:-

File Structure:

```
project-root/
├── Jenkinsfile
├── ansible/
│   ├── inventory
│   └── playbook.yml
```

```groovy
pipeline {
  agent any
  tools {
    ansible 'ansible' // name configured in Jenkins > Global Tool Configuration
  }
  stages {
    stage('Run Ansible Playbook') {
      steps {
        ansiblePlaybook(
          credentialsId: 'ssh',
          disableHostKeyChecking: true,
          installation: 'ansible',
          inventory: 'ansible/inventory',
          playbook: 'ansible/playbook.yml'
        )
      }
    }
  }
}
```

```ini
ansible/inventory:
[appserver]
12.34.56.78 ansible_user=ec2-user
```

credentialsId: 'ssh'
This refers to SSH credentials added to Jenkins under Manage Jenkins ➝ Credentials.

If your YAML file is at ansible/k8s_deployments.yaml, Jenkins will now have it at:
$WORKSPACE/ansible/k8s_deployments.yaml

2. Use Ansible copy Module to Transfer File

In your Ansible playbook, just use the local path relative to Jenkins workspace as src:.

YAML

```yaml
- name: deploy to eks cluster
  hosts: appserver
  gather_facts: no
  become: yes
  tasks:
    - name: copy k8s deployment YAML file to remote EC2
      copy:
        src: ansible/k8s_deployments.yaml  # Path relative to Jenkins working directory
        dest: /home/ec2-user/k8s_deployments.yaml
        mode: '0644'
```

## Integrate Jenkins with Kubernetes

First install two plugins Kubernetes and Kubernetes cli

Then create a service account in the Kubernetes cluster and deploy with that service account

First create a namespace Jenkins

```bash
kubectl create ns jenkins
```

Creating Service Account and Create Role and Bind the role to service account

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: jenkins
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: jenkins
rules:
  - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: jenkins 
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role 
subjects:
- namespace: jenkins 
  kind: ServiceAccount
  name: jenkins 
```

Now to use service account in Jenkins we need to create a token for authentication

Apply the below file

```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: jenkins
```

```bash
kubectl apply -f filename -n jenkins  # specially mention so it creates in Jenkins namespace 

kubectl describe secret mysecretname -n jenkins  # to get the token 
```

And copy the token and create a secret text credential of this token

Now go to pipeline syntax and search for withKubeCredentials 

And give the token and endpoint can be found on aws eks dashboard and name and give namespace

```groovy
withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: ' activity-eks-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'jenkins', serverUrl: 'https://677E6481C9881719E5DEA58BCFCDABE9.gr7.us-east-1.eks.amazonaws.com']]) {
    // some block
}
```

And give commands in the block

```groovy
stage('destroy the k8s') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'activity-eks-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'jenkins', serverUrl: 'https://59F730EEB80A4B34FD933ECA2E56EAC8.gr7.us-east-1.eks.amazonaws.com']]) {
                    script {
                        def releaseExists = sh(
                            script: "helm list -n jenkins -q | grep -w nginx || echo 'not found'",
                            returnStdout: true
                        ).trim()
                        
                        if (releaseExists == "nginx") {
                            sh "helm uninstall nginx -n jenkins"
                        } else {
                            echo "Helm release 'nginx' does not exist. Skipping uninstallation."
                        }
                    }
                }
            }
        }
        stage('edit the helm-chart') {
            steps {
                withCredentials([string(credentialsId: 'github_token', variable: 'github_token')]) {
                    sh '''
                        git config user.email "santoshpalla2002@gmail.com"
                        git config user.name "santoshpalla27"
                        sed -i "s#santoshpalla27/nginx:v.*#santoshpalla27/nginx:v${BUILD_NUMBER}#" helm-chart/values.yaml
                        git add .
                        git commit -m "Updated Docker images to version ${BUILD_NUMBER}"
                        git push https://${github_token}@github.com/santoshpalla27/activity.git HEAD:main
                        '''
                }
            }
        }

        
        stage('deploy to k8s') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'activity-eks-cluster', contextName: '', credentialsId: 'k8s-token', namespace: 'jenkins', serverUrl: 'https://59F730EEB80A4B34FD933ECA2E56EAC8.gr7.us-east-1.eks.amazonaws.com']]) {
                    sh "helm install nginx ./helm-chart -n jenkins"
                }
            }
        }
```

Or also deploy using Jenkins agent which has access to the eks cluster using kubeconfig 

```bash
sh '''
    # Update kubeconfig
    aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
    
    # Verify connection
    kubectl get nodes
'''
```

```groovy
pipeline {
    agent {
        label 'eks-agent'  // Agent that has kubectl and kubeconfig configured
    }
    
    stages {
        stage('Deploy to K8s using kubeconfig') {
            steps {
                script {
                    // Set the kubeconfig if it's in a custom location
                    sh '''
                        export KUBECONFIG=/path/to/kubeconfig
                        
                        # Verify connection
                        kubectl get nodes
                        
                        # Apply your kubernetes manifests
                        kubectl apply -f k8s-manifests/ -n jenkins
                        
                        # Or use Helm
                        helm upgrade --install nginx ./helm-chart -n jenkins
                    '''
                }
            }
        }
    }
}
```

## Docker Agents

Custom Docker Agent for Helm + kubectl (GitOps-ready)

Create a custom image like:

```dockerfile
FROM alpine:3.18

RUN apk add --no-cache \
    bash curl git openssh \
    kubectl helm

WORKDIR /app
ENTRYPOINT ["/bin/sh"]
```

Tag and push to your Docker registry, then use:

```groovy
agent {
    docker {
        image 'myregistry.io/jenkins-helm-kubectl:latest'
    }
}
```

```groovy
pipeline {
    agent {
        docker {
            image 'myregistry.io/jenkins-helm-kubectl:latest'
        }
    }

    stages {
        stage('K8s check') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    credentialsId: 'k8s-token',
                    clusterName: 'activity-eks-cluster',
                    namespace: 'jenkins',
                    serverUrl: 'https://my.eks.amazonaws.com'
                ]]) {
                    sh 'kubectl get pods -n jenkins'
                }
            }
        }
    }
}
```

### Multi Agent Pipeline

Yes, ✅ you can absolutely use multiple agents in a single Jenkins pipeline using Declarative Pipeline syntax — either by:

```groovy
pipeline {
    agent none  // No global agent — define per stage

    stages {
        stage('Build Java') {
            agent {
                docker {
                    image 'maven:3.9.3-eclipse-temurin-21'
                }
            }
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Lint Python') {
            agent {
                docker {
                    image 'python:3.11'
                }
            }
            steps {
                sh 'pip install flake8 && flake8 app/'
            }
        }

        stage('Deploy to K8s') {
            agent {
                label 'eks-agent'  // Static Jenkins agent with kubeconfig
            }
            steps {
                sh 'kubectl get pods -n myns'
            }
        }
    }
}
```

### How to share data (artifacts, files, manifests, Helm charts, Ansible playbooks) between stages that run on different agents?

Each Jenkins agent runs in its own workspace (especially if they're on different nodes or Docker containers). Unless you handle it properly, your build outputs (compiled code, Dockerfiles, helm charts, etc.) won't exist across stages.

**Solution: Use stash and unstash to share files between stages**
Jenkins provides stash and unstash to pass files/data between stages (even across different agents or agent types).

```groovy
pipeline {
    agent none  // We define agent per stage

    stages {
        stage('Build Node.js App') {
            agent { label 'node-agent' }

            steps {
                sh 'npm install'
                sh 'npm run build'

                // Archive build artifacts (e.g., dist/, Dockerfile, manifests)
                stash includes: '**/*', name: 'app-build'
            }
        }

        stage('Docker Build & Push') {
            agent { label 'docker-agent' }

            steps {
                unstash 'app-build'

                sh '''
                    docker build -t myregistry.io/myapp:${BUILD_NUMBER} .
                    docker push myregistry.io/myapp:${BUILD_NUMBER}
                '''

                // Stash again if you want to preserve manifests for deployment
                stash includes: '**/*.yaml', name: 'k8s-manifests'
            }
        }

        stage('Deploy with Kubernetes Agent') {
            agent { label 'k8s-agent' }

            steps {
                unstash 'k8s-manifests'

                withKubeCredentials(kubectlCredentials: [[
                    credentialsId: 'k8s-token',
                    clusterName: 'activity-eks-cluster',
                    namespace: 'default',
                    serverUrl: 'https://abcd.eks.amazonaws.com'
                ]]) {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }

        stage('Deploy with Ansible Agent') {
            agent { label 'ansible-agent' }

            steps {
                unstash 'app-build'  // Playbooks/roles can also be stashed earlier, if needed

                sh '''
                    ansible-playbook -i inventory.yaml playbook.yaml
                '''
            }
        }
    }
}
```

### Understanding stash / unstash

| Function | Behavior |
|----------|----------|
| `stash name: 'x', includes: '**/*.yaml'` | Zips and stores matching files from current workspace |
| `unstash 'x'` | Extracts those files to current workspace of another stage/agent |

Jenkins downloads source code only in the workspace where checkout scm is called.
If later stages:
- Use the same workspace and agent → ✅ source code persists.
- Use a different agent (Docker/container/another node) → ❌ the workspace is new, so Git repo isn't present unless you do another checkout.

### Ways to Handle It

✅ 1. Use stash/unstash to pass Git repo between stages

```groovy
stage('Checkout') {
    agent { label 'lightweight' }
    steps {
        checkout scm
        stash name: 'source', includes: '**'
    }
}

stage('Build') {
    agent { label 'builder' }
    steps {
        unstash 'source'
        sh 'npm run build'
    }
}
```

## Edit Manifests in GitHub

```groovy
withCredentials([string(credentialsId: 'github_token', variable: 'github_token')]) {
    sh '''
        git config user.email santoshpalla2002@gmail.com"
        git config user.name santoshpalla27"
        sed -i "s#santoshpalla27/nginx:v.*#santoshpalla27/nginx:v${BUILD_NUMBER}#" helm-chart/values.yaml
        git add .
        git commit -m "Updated Docker images to version ${BUILD_NUMBER}"
        git push https://${github_token}@github.com/santoshpalla27/activity.git HEAD:main
        '''
}
```

Use pipeline syntax to bind credentials 

Create a GitHub token by going to GitHub setting developer setting and create a classic token and create a credential with name github_token

Change image name and repo name

## Deploy to EC2 Instance using SSH

Create a credential as ssh-token with ssh username and private key and then 

```bash
sudo usermod -aG docker ec2-user  # run this on the remote server
```

```groovy
stage('deploy to ec2 instance'){
            steps{
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ssh-token', keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ec2-user@34.226.140.215 '
                                docker pull ${FRONTEND_IMAGE} || true
                                docker stop frontend-image || true
                                docker rm frontend-image || true
                                docker run -d --name frontend-image -p 80:80 ${FRONTEND_IMAGE}
                            '
                        """
                    }
                }
           }
        }
```

## Post Build Actions

Used to send notification or alert or email for the build is a success of failure 

For this to happen we have to generate a password for the gmail go to https://myaccount.google.com/apppasswords

And create a password for Jenkins vleb jtsx evjj djhl

Also open port 465 in Jenkins server

Go to manage Jenkins -> system and email notification and in smtp server enter smtp.gmail.com 

Go to advanced use smtp authentication and give username as email and password token and enable ssl and smtp port 465

Do the same for extended smtp and in credentials give gmail and token 

```groovy
post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
            <html>
            <body>
            <div style="border: 4px solid ${bannerColor}; padding: 10px">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                    <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${env.BUILD_URL}">console output</a>.</p>
            </div>
            </body>
            </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'jaiswaladi246@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
            )
        }
    }
}
```

## Jenkins Slack Notification

1. Create a Slack App / Token
Go to: https://api.slack.com/apps

Create a new app → From scratch

Enable Incoming Webhooks in features

Add a new Webhook URL for the desired Slack channel

Copy the Webhook URL or generate a Bot Token if using the Slack plugin with credentials

https://hooks.slack.com/services/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

xoxb-9043736372785-9024837805667-5ZqAMi8Yiyxr3CGHvd4rgzv4

Also go to oauth and permissions and copy bot user token

2. Install Slack Notification Plugin in Jenkins

Go to Manage Jenkins → Manage Plugins

Search for and install: Slack Notification Plugin

3. Configure Slack in Jenkins

Go to Manage Jenkins → Configure System

Find the Slack section

Fill in:

Workspace  (name of workspace)

Integration Token Credential ID (add secret token or webhook as Jenkins credential)  use bot user token

Default channel  or channel id which can be found on slack channel details by scrolling down on slack view channel details

And check the custom slack app bit user

Test connection to verify

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
    }

    post {
        success {
            slackSend(channel: '#devops-alerts', message: "✅ Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' succeeded.")
        }
        failure {
            slackSend(channel: '#devops-alerts', message: "❌ Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' failed.")
        }
    }
}
```

Use pipeline syntax 

```groovy
slackSend channel: '#aws-cost-report', message: 'hello from jenkins'
```

## Parameters in CI/CD

```groovy
pipeline {
    agent any

    parameters {
        string(name: 'STRING_PARAM', defaultValue: 'Hello', description: 'Enter a string value')
        booleanParam(name: 'BOOLEAN_PARAM', defaultValue: true, description: 'Check for true/false')
        choice(name: 'CHOICE_PARAM', choices: ['dev', 'qa', 'prod'], description: 'Pick an environment')
        password(name: 'PASSWORD_PARAM', defaultValue: '', description: 'Enter a password (hidden)')
        text(name: 'TEXT_PARAM', defaultValue: 'This is a\nmulti-line\ntext input', description: 'Multi-line text')
        file(name: 'FILE_PARAM', description: 'Upload a file (used in freestyle jobs)')
        // credentials can only be used in certain plugins or contexts
        credentials(name: 'CRED_PARAM', description: 'Select Jenkins credentials')
    }

    stages {
        stage('Show Parameters') {
            steps {
                echo "STRING_PARAM: ${params.STRING_PARAM}"
                echo "BOOLEAN_PARAM: ${params.BOOLEAN_PARAM}"
                echo "CHOICE_PARAM: ${params.CHOICE_PARAM}"
                echo "TEXT_PARAM: ${params.TEXT_PARAM}"
                echo "PASSWORD_PARAM: ${params.PASSWORD_PARAM}" // not safe to print in real use
                echo "FILE_PARAM: ${params.FILE_PARAM}" // only works in certain contexts
                echo "CRED_PARAM: ${params.CRED_PARAM}" // ID of selected Jenkins credentials
            }
        }
    }
}
```

Solution: Trigger a Dummy First Build
- Create your pipeline job.
- Paste the pipeline code with the parameters block.
- Run the first build (it may ignore parameters).
- Stop or let the first build finish.
- Now go to Build with Parameters – parameters will appear and work from the second build onward.

#### 1. string
🧩 Use Case: Provide a custom application version or feature branch name to build/deploy.

```groovy
parameters {
    string(name: 'APP_VERSION', defaultValue: '1.0.0', description: 'Application version to build')
}

steps {
    sh "echo Deploying version ${params.APP_VERSION}"
}
```

#### 2. booleanParam
🧩 Use Case: Toggle whether to run integration tests or deploy to production.

```groovy
parameters {
    booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run integration tests?')
}

steps {
    script {
        if (params.RUN_TESTS) {
            echo "Running integration tests..."
            // sh './run-tests.sh'
        } else {
            echo "Skipping tests"
        }
    }
}
```

#### 3. choice
🧩 Use Case: Select the target environment (dev, qa, staging, prod) for deployment.

```groovy
parameters {
    choice(name: 'ACTION', choices: ['plan', 'apply', 'destroy'], description: 'Choose Terraform action')
}
```

```groovy
stage('Apply or Destroy Terraform') {
            when {
                expression { params.ACTION == 'apply' || params.ACTION == 'destroy' }
            }
            steps {
                dir(env.TF_WORKING_DIR) {
                    script {
                        if (params.ACTION == 'apply') {
                            sh 'terraform apply -auto-approve tfplan'
                        } else if (params.ACTION == 'destroy') {
                            sh 'terraform destroy -auto-approve'
                        }
                    }
                }
            }
        }
```

## Parallel Execution in Jenkins

How do you implement parallel execution in Jenkins?
Answer:
Parallel execution in Jenkins can be implemented using the parallel step in a pipeline.

```groovy
pipeline {
    agent any

    stages {
        stage('Parallel Stage') {
            parallel {
                stage('Build') {
                    steps {
                        echo 'Building...'
                    }
                }
                stage('Test') {
                    steps {
                        echo 'Testing...'
                    }
                }
            }
        }
    }
}
```

## Stash and Unstash Steps in Jenkins Pipeline

What is the use of the stash and unstash steps in Jenkins Pipeline?
Answer:

stash and unstash steps in Jenkins Pipeline are used to share files between stages, especially when those stages run on different agents or workspaces.

They allow you to preserve and transfer workspace contents (e.g., build artifacts, source code, configuration files) from one stage to another.

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'make'
                stash includes: '**/target/*.jar', name: 'build-artifacts'
            }
        }

        stage('Test') {
            steps {
                unstash 'build-artifacts'
                sh 'make test'
            }
        }
    }
}
```

How to share data (artifacts, files, manifests, Helm charts, Ansible playbooks) between stages that run on different agents?

Each Jenkins agent runs in its own workspace (especially if they're on different nodes or Docker containers). Unless you handle it properly, your build outputs (compiled code, Dockerfiles, helm charts, etc.) won't exist across stages.

**Solution: Use stash and unstash to share files between stages**
Jenkins provides stash and unstash to pass files/data between stages (even across different agents or agent types).

```groovy
pipeline {
    agent none  // We define agent per stage

    stages {
        stage('Build Node.js App') {
            agent { label 'node-agent' }

            steps {
                sh 'npm install'
                sh 'npm run build'

                // Archive build artifacts (e.g., dist/, Dockerfile, manifests)
                stash includes: '**/*', name: 'app-build'
            }
        }

        stage('Docker Build & Push') {
            agent { label 'docker-agent' }

            steps {
                unstash 'app-build'

                sh '''
                    docker build -t myregistry.io/myapp:${BUILD_NUMBER} .
                    docker push myregistry.io/myapp:${BUILD_NUMBER}
                '''

                // Stash again if you want to preserve manifests for deployment
                stash includes: '**/*.yaml', name: 'k8s-manifests'
            }
        }

        stage('Deploy with Kubernetes Agent') {
            agent { label 'k8s-agent' }

            steps {
                unstash 'k8s-manifests'

                withKubeCredentials(kubectlCredentials: [[
                    credentialsId: 'k8s-token',
                    clusterName: 'activity-eks-cluster',
                    namespace: 'default',
                    serverUrl: 'https://abcd.eks.amazonaws.com'
                ]]) {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }

        stage('Deploy with Ansible Agent') {
            agent { label 'ansible-agent' }

            steps {
                unstash 'app-build'  // Playbooks/roles can also be stashed earlier, if needed

                sh '''
                    ansible-playbook -i inventory.yaml playbook.yaml
                '''
            }
        }
    }
}
```

### Understanding stash / unstash

| Function | Behavior |
|----------|----------|
| `stash name: 'x', includes: '**/*.yaml'` | Zips and stores matching files from current workspace |
| `unstash 'x'` | Extracts those files to current workspace of another stage/agent |

Jenkins downloads source code only in the workspace where checkout scm is called.
If later stages:
- Use the same workspace and agent → ✅ source code persists.
- Use a different agent (Docker/container/another node) → ❌ the workspace is new, so Git repo isn't present unless you do another checkout.

### Ways to Handle It
✅ 1. Use stash/unstash to pass Git repo between stages

```groovy
stage('Checkout') {
    agent { label 'lightweight' }
    steps {
        checkout scm
        stash name: 'source', includes: '**'
    }
}

stage('Build') {
    agent { label 'builder' }
    steps {
        unstash 'source'
        sh 'npm run build'
    }
}
```

## Input Step in Jenkins Pipeline

What is the input step in Jenkins Pipeline?
Answer:
The input step pauses the pipeline and waits for human input before proceeding.

The input step pauses the pipeline and waits for human input (such as a manual approval) before proceeding to the next stage. This is commonly used for manual approvals before deploying to production.

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }

        stage('Approval') {
            steps {
                input message: 'Do you want to proceed to deployment?', ok: 'Yes, Deploy'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
```

## Integrating Jenkins with AWS

How do you integrate Jenkins with AWS?
Answer:
• Use AWS plugins like AWS CodeDeploy, AWS S3, and AWS EC2.
• Configure AWS credentials in Jenkins (using credentials manager or environment variables).
• Use AWS build steps in your pipeline to deploy applications and manage AWS resources.

```groovy
pipeline {
    agent any

    stages {
        stage('Upload to S3') {
            steps {
                s3Upload(bucket: 'my-bucket', path: 'myapp.jar')
            }
        }

        stage('Deploy with CodeDeploy') {
            steps {
                codedeployDeploy(
                    applicationName: 'MyApp',
                    deploymentGroupName: 'MyDeploymentGroup',
                    s3Bucket: 'my-bucket',
                    s3Key: 'my-app.jar'
                )
            }
        }
    }
}
```

## Retry Step in Jenkins Pipeline

What is the retry step in Jenkins Pipeline?
Answer:
The retry step retries a block of steps a specified number of times in case of failure.

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                retry(3) {
                    sh 'make build'
                }
            }
        }
    }
}
```

## Deploy to ASG Instances

```groovy
stage('deploy to asg instances') {
            steps {
                script {
                    withCredentials([
                        sshUserPrivateKey(credentialsId: 'ssh-token', keyFileVariable: 'SSH_KEY'),
                        string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY'),
                        string(credentialsId: 'aws-region', variable: 'AWS_REGION')
                    ]) {
                        def instanceIps = sh(script: '''
                            export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                            export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                            export AWS_REGION=${AWS_REGION}

                            aws ec2 describe-instances \
                            --filters "Name=tag:Name,Values=web-server" "Name=instance-state-name,Values=running" \
                            --query "Reservations[*].Instances[*].PublicIpAddress" \
                            --output text
                        ''', returnStdout: true).trim().split()

                        if (instanceIps.size() == 0) {
                            error "No EC2 instances found with the tag Name=web-server"
                        }

                        for (ip in instanceIps) {
                            echo "Deploying to ${ip}"
                            sh """
                                ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ec2-user@${ip} '
                                    docker pull ${FRONTEND_IMAGE} || true
                                    docker stop frontend-image || true
                                    docker rm frontend-image || true
                                    docker run -d --name frontend-image -p 80:80 ${FRONTEND_IMAGE}
                                '
                            """
                        }
                    }
                }
            }
        }
```