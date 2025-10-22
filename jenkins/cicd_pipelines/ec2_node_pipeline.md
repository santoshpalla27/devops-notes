# EC2 Node.js Pipeline

```groovy
pipeline {
    agent any
    tools {
        nodejs 'nodejs'
    }
    environment  {
        SCANNER_HOME=tool 'sonar-scanner'  
        FRONTEND_IMAGE = "santoshpalla27/sample:frontend-${BUILD_NUMBER}"  
    }
    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/santoshpalla27/mern-employee-docker-compose.git'
            }
        }
        stage('Build') {
            steps {
                dir('mern/frontend') {
                    sh 'npm install'
                    // sh 'npm run build'
                }
            }
        }
        stage('sonar scan') {
            steps {
                dir('mern/frontend') {
                    withSonarQubeEnv('sonar-server') {   
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=sample-frontend \
                        -Dsonar.projectKey=sample-frontend \
                        -Dsonar.sources=. '''   
                    }
                }
            }
        }
        stage('Quality Gate frontend') {
            steps {
                script {
                    timeout(time: 1, unit: 'MINUTES') {  
                        def qualityGate = waitForQualityGate()  
                        if (qualityGate.status != 'OK') {  
                            error "Quality Gate failed: ${qualityGate.status}"
                        } else {
                            echo "Quality Gate passed"
                        }
                    }
                }
            }
        }
        stage('building frontend image') {
            steps {
                dir('mern/frontend'){
                    script {
                        sh "docker build -t ${FRONTEND_IMAGE} -f Dockerfile ."
                    }
                }
            }
        }
        stage('trivy scan frontend-image'){
            steps {
                script {
                    sh "trivy image ${FRONTEND_IMAGE} > trivy-report.txt"
                }
            }
        }
        stage('image push to docker hub') {
            steps {
                dir('mern/frontend'){
                    script {
                        withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker push ${FRONTEND_IMAGE}"
                        }  
                    }
                }
            }
        }
        stage('deploy to single instance'){
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

    }
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
                    to: 'santoshpalla27@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'santoshpalla27@example.com',
                    mimeType: 'text/html',
                )
            }           
            sh 'docker system prune -f || true'
        }
   }    
}
```