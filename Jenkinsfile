pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Hello from Jenkins!'
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t devops-web:v1 .'
            }
        }
    }
}
