pipeline {
    agent any
    tools{
        maven 'maven3'
        jdk 'jdk17'
    }
    stages {
        stage('Cloning Git') {
            steps {
                script {
                    echo "Cloning Git..."
                    git branch: "${params.branch}", credentialsId: 'github', url: "${params.url}"
                    stash includes: '**', name: "stash-${params.pname}"
                }
            }
        }
        stage('Building') {
            steps {
                script {
                    dir("run-${params.pname}-${BUILD_NUMBER}") {
                        echo "Building..."
                        unstash "stash-${params.pname}"
                        bat "tree /f"
                        bat "mvn clean package"
                    }
                }
            }
        }
        stage('Testing') {
            steps {
                script {
                    dir("run-${params.pname}-${BUILD_NUMBER}") {
                        echo "Testing..."
                        unstash "stash-${params.pname}"
                        bat "tree /f"
                        bat "mvn test"
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                dir("run-${params.pname}-${BUILD_NUMBER}") {
                    echo "Cleaning up..."
                    deleteDir() 
                }   
               
            }
        }
    }
}
