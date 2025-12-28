pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'SonarScanner'
        ACR_LOGIN_SERVER = 'devopsproject1.azurecr.io'
        IMAGE_NAME = 'taskpro'
        TAG = 'latest'
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build the code') {
            steps {
                // Tumchya pom.xml madhye Java 11 ahe, mhanun ithe Maven build suru hoil
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=task-master-pro \
                    -Dsonar.sources=.
                    """
                }
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                dependencyCheck additionalArguments: '--scan pom.xml', odcInstallation: 'Dependency-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Building Docker image'){
            steps {
                // Image build kartaana tag ithech dila tar bare padte
                sh "docker build -t ${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Login to ACR') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'acr-creds',
                    usernameVariable: 'ACR_USER',
                    passwordVariable: 'ACR_PASS'
                )]) {
                    sh "echo ${ACR_PASS} | docker login ${ACR_LOGIN_SERVER} -u ${ACR_USER} --password-stdin"
                }
            }
        }

        stage('Tag and Push to ACR') {
            steps {
                sh """
                docker tag ${IMAGE_NAME}:${TAG} ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${TAG}
                docker push ${ACR_LOGIN_SERVER}/${IMAGE_NAME}:${TAG}
                """
            }
        }
    }
}