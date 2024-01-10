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
                    dir("run-${BUILD_NUMBER}") {
                        echo "Building..."
                        bat "tree /f"
                        unstash "stash-${params.pname}"
                        sh "mvn clean package"
                    }
                }
            }
        }
        stage('Testing') {
            steps {
                script {
                    dir("run-${BUILD_NUMBER}") {
                        echo "Testing..."
                        bat "tree /f"
                        unstash "stash-${params.pname}"
                        sh "mvn test"
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                dir("run-${BUILD_NUMBER}") {
                    echo "Cleaning up..."
                    deleteDir() 
                }   
               
            }
        }
    }
}
