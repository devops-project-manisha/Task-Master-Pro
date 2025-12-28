pipeline {
    agent any

    parameters {
        choice(name: 'ENV', choices: ['Dev', 'QA', 'Prod'], description: 'Select Environment')
    }

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

        stage('Building Docker image') {
            steps {
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

        stage('Deploy the docker image to QA server') {
            when {
                expression {
                    params.ENV == 'QA'
                }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'acr-creds',
                    usernameVariable: 'ACR_USER',
                    passwordVariable: 'ACR_PASS'
                )]) {
                    sh '''
                    ssh jenkins@4.222.234.133 \
                    ansible-playbook /home/jenkins/Myansible/masterpro.yml \
                    -e acr_username=$ACR_USER \
                    -e acr_password=$ACR_PASS \
                    -b
                    '''
                }
            }
        }
    }
}
