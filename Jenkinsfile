pipeline {
    agent {
        label 'agent'
    }
    tools {
        jdk 'jdk17'
        nodejs 'node22'
    }
    environment {
        //SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "mytodoapp"
        RELEASE = "1.0.0"
        DOCKER_USER = "georgeao"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        DOCKER_REGISTRY_CREDENTIALS = 'docker' // Credentials ID for Docker registry
        GIT_REPO_NAME = "todolistapp-k8s"
        GIT_USER_NAME = "kayodeao"
    }
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/kayodeao/todolistapp.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        
        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: DOCKER_REGISTRY_CREDENTIALS, toolName: 'docker') {
                        sh "docker build  -t ${IMAGE_NAME} ."
                        sh "docker tag ${IMAGE_NAME} ${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git clone https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git
                        cd ${GIT_REPO_NAME}
                        git config user.email "kayode@aopartnersng.com"
                        git config user.name "${GIT_USER_NAME}"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        sed -i "s|image: ${DOCKER_USER}/${APP_NAME}:.*|image: ${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}|g" kubernetes/deployment.yaml
                        git add kubernetes/deployment.yaml
                        git commit -m "Updated deployment image to version ${RELEASE}-${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }
    post {
        always {
            emailext (
                subject: "Build Status: ${currentBuild.currentResult}",
                body: """Build Details:
                         - Job Name: ${env.JOB_NAME}
                         - Build Number: ${env.BUILD_NUMBER}
                         - Build Status: ${currentBuild.currentResult}
                         - Build URL: ${env.BUILD_URL}

                         """,
                to: 'george@aopartners.io',
                //attachmentsPattern: 'trivyfs.txt, trivyimage.txt'
            )
        }
    }
}
