# Node.js Pipeline

```groovy
pipeline {
    agent any
    tools {
        nodejs 'nodejs'  // this should match the tools installation configuration name like nodejs is default for nodejs tool and 'nodejs' is name that should match
    }
    environment  {
        SCANNER_HOME=tool 'sonar-scanner'  // this should match the sonar-scanner name configured in tools
        FRONTEND_IMAGE = "santoshpalla27/3-tier:frontend-${BUILD_NUMBER}" // need to use double quotes to use variables
        BACKEND_IMAGE = "santoshpalla27/3-tier:backend-${BUILD_NUMBER}"  // need to use double quotes to use variables
    }   
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()  // Cleans the workspace
            }
        }
        stage('poll scm') {
            steps {
                git branch: 'main', url: 'https://github.com/santoshpalla27/mern-employee-app.git'
            }
        }
        stage('trivy scan') {
            steps {
                sh ' trivy fs . > trivy-report.txt'
            }
        }
        stage('sonar scan') {
            steps {
                dir('mern/frontend'){
                    withSonarQubeEnv('sonar-server') {   // sonar-server should match with the name configure in the system configuration
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=frontend \
                        -Dsonar.projectKey=frontend \
                        -Dsonar.sources=. '''   // this will scan all the directories including the sub directories
                    }
                }
            }
        } 
        stage('Quality Gate frontend') {
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
        stage('sonar scan backend') {
            steps {
                dir('mern/backend'){
                    withSonarQubeEnv('sonar-server') {   // sonar-server should match with the name configure in the system configuration
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=backend \
                        -Dsonar.projectKey=backend \
                        -Dsonar.sources=. '''   // this will scan all the directories including the sub directories
                    }
                }
            }
        }
        stage('Quality Gate backend') {
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
        stage('remove previous images') {
            steps {
                script {
                    // Check if there are any Docker images
                    def images = sh(script: "docker images -q", returnStdout: true).trim()
                    
                    // If there are images, run docker rmi to remove them
                    if (images) {
                        sh "docker rmi -f ${images}"
                    } else {
                        echo "No Docker images to remove."
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
                    sh "trivy image ${FRONTEND_IMAGE} > frontend-trivy-report.txt"
                }
            }
        } 
        stage(']building backend image') {
            steps {
                dir('mern/backend'){
                    script {
                        sh "docker build -t ${BACKEND_IMAGE} -f Dockerfile ."
                    }
                }
            }
        }
        stage('trivy scan backend-image'){
            steps {
                script {
                    sh "trivy image ${BACKEND_IMAGE} > backend-trivy-report.txt"
                }
            }
        } 
        stage('docker push') {
           steps {
               withCredentials([string(credentialsId: 'docker-pass', variable: 'DOCKER_PASS')]) {  //var should not contain - instead use _
                  script {
                      sh "docker login -u santoshpalla27 -p ${DOCKER_PASS}"
                      sh "docker push ${FRONTEND_IMAGE}"
                      sh "docker push ${BACKEND_IMAGE}"
                    }
                }
            }
        }
        stage('update github'){
            environment {
                    GITHUB_USERNAME = "santoshpalla27"
                    GITHUB_EMAIL = "santoshpalla2002@gmail.com"
                    GITHUB_REPO = "mern-employee-app"
               }
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "${GITHUB_EMAIL}"
                        git config user.name "${GITHUB_USERNAME}"
                        sed -i "s#santoshpalla27/3-tier:frontend.*#santoshpalla27/3-tier:frontend-${BUILD_NUMBER}#" kubernetes-manifest/frontend-deployment.yaml
                        sed -i "s#santoshpalla27/3-tier:backend.*#santoshpalla27/3-tier:backend-${BUILD_NUMBER}#" kubernetes-manifest/backend.yaml
                        git add .
                        git commit -m "Updated Docker images to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GITHUB_USERNAME}/${GITHUB_REPO}.git HEAD:main
                        '''
                }    
            }
        }
    }
}
```