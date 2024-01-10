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
                        bat "mvn clean package -DskipTests=true"
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
                        bat "mvn test"
                    }
                }
            }
        }
        stage('Dependency Check') {
            steps {
                script {
                    dir("run-${params.pname}-${BUILD_NUMBER}") {
                        echo "Dependency Check..."
                        unstash "stash-${params.pname}"
                        bat "mvn org.owasp:dependency-check-maven:check"
                    }
                }
            }
        }
        stage('Building Docker Image') {
            steps {
                script {
                    dir("run-${params.pname}-${BUILD_NUMBER}") {
                        echo "Deploying..."
                        unstash "stash-${params.pname}"
                        bat "docker build -t ${params.pname}:${BUILD_NUMBER} ."
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
