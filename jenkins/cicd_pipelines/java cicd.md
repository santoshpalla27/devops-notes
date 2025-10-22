# Java CI/CD Pipeline

```groovy
pipeline {
    agent any
    tools {
        maven 'maven'
        jdk 'jdk'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage("clean workspace") {
            steps {
                cleanWs()
            }
        }
        stage('poll scm') {
            steps {
                git branch: 'main', url: 'https://github.com/santoshpalla27/java-cicd.git'
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('trivy fs scan') {
            steps {
                sh 'trivy fs --format table -o fs.html .'
            }
        }
        stage('sonar scan') {
            steps {
                withSonarQubeEnv('sonar-server') { 
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=santosh-java \
                        -Dsonar.projectKey=santosh-java \
                        -Dsonar.sources=. '''   
                }
            }
        }
        stage('sonar quality gate') {
            steps {
                script {
                    def qualityGate = waitForQualityGate()  
                    if (qualityGate.status != 'OK') {  
                        error "Quality Gate failed: ${qualityGate.status}"
                    }
                    else{
                        echo "Quality Gate passed"
                    }
                }
            }
        }
        stage('build') {
            steps {
                sh 'mvn clean install '
            }
        }
        stage('nexus upload') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-setting', jdk: 'jdk', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        stage('owasp dependency check') {
            steps {
                withCredentials([string(credentialsId: 'owasp', variable: 'NVD_API_KEY')]) {
                    dependencyCheck additionalArguments: "--scan target/ --nvdApiKey ${NVD_API_KEY}", odcInstallation: 'owasp'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        stage('remove previous images'){
            steps {
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
            }
        }
        stage('docker build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t santoshpalla27/java-cicd:${BUILD_NUMBER} .'
                    }
                }
            }
        }
        stage('trivy image scan') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'trivy image --format table -o image.html santoshpalla27/java-cicd:${BUILD_NUMBER}'
                    }
                }
            }
        }
        stage('docker push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push santoshpalla27/java-cicd:${BUILD_NUMBER} '                    
                    }
                }
            }
        }
        stage('edit kubernetes manifests'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'github_token', variable: 'github_token')]) {
                        sh '''
                            git config user.email "santoshpalla2002@gmail.com"
                            git config user.name "santoshpalla27"
                            sed -i "s#santoshpalla27/*.*#santoshpalla27/java-cicd:${BUILD_NUMBER}#" deployment/tomcat-deployment.yaml
                            git add .
                            git commit -m "Updated Docker images to version ${BUILD_NUMBER}"
                            git push https://${github_token}@github.com/santoshpalla27/java-cicd.git HEAD:main

                        '''
                    }
                }
            }
        }
        stage('k8s deploy') {
            steps {    
                script {
                    withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'santosh', contextName: '', credentialsId: 'k8s-token', namespace: 'jenkins', serverUrl: 'https://5EFFC5127857BA1BA505B4BEED3EC976.gr7.us-east-1.eks.amazonaws.com']]) {
                        sh 'kubectl apply -f deployment/tomcat-deployment.yaml'
                    }
                }
            }
        }
        stage('k8s verify deployment') {
            steps {    
                script {
                    withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'santosh', contextName: '', credentialsId: 'k8s-token', namespace: 'jenkins', serverUrl: 'https://5EFFC5127857BA1BA505B4BEED3EC976.gr7.us-east-1.eks.amazonaws.com']]) {
                        sh '''
                            kubectl get svc -n jenkins
                            kubectl get pods -n jenkins
                        '''
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
        }
   }
}
```