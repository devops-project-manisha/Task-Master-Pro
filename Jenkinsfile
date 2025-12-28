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
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SCANNER_HOME = tool 'SonarScanner'
            }
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
        steps{
          sh "docker build -t taskpro:latest ."
        }

       }

       stage('Login to ACR') {
       steps {
         withCredentials([usernamePassword(
             credentialsId: 'acr-creds',
             usernameVariable: 'ACR_USER',
             passwordVariable: 'ACR_PASS'
         )]) {
             sh '''
               echo $ACR_PASS | docker login $ACR_LOGIN_SERVER \
               -u $ACR_USER --password-stdin
             '''
           }
          }
         }

         stage('Tag name') {
            steps {
                sh '''
                  docker tag ${IMAGE_NAME}:$ {TAG} \
                  $ACR_LOGIN_SERVER/${IMAGE_NAME}: ${TAG}
                '''
            }
         }
         

         


    }
}
