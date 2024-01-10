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
                    echo "Building..."
                    unstash "stash-${params.pname}"
                    sh "mvn clean install"
                }
            }
        }
    }
}
