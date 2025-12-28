pipeline {
    agent any

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
          sh "docker build -t taskpro:latest ."

       }



    }
}
