pipeline {
    agent any
    parameters {
        string(name: 'SERVICE_NAME', description: 'Name of the microservice')
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building ${params.SERVICE_NAME}..."
                }
            }
        }
    }
}
