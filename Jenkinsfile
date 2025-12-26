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
    }
}


