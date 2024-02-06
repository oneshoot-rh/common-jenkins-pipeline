pipeline {
    agent any
    tools{
        maven 'maven3'
        jdk 'jdk17'
    }
    environment {
        DIR_NAME = "run-${params.pname}-${params.BUILD_NUMBER}"
        DEPLOYENV = 'NONE'
        BRANCH = "${params.branch}"
    }
    stages {
        stage('Cloning Git') {
            steps {
                script {
                    echo "Cloning Git..."
                    echo "branch name: ${BRANCH}"
                    git branch: "${BRANCH}", credentialsId: 'github', url: "${params.url}"
                    stash includes: '**', name: "stash-${params.pname}"
                }
            }
        }
        stage('Checkout ENV'){
            steps{
                script{
                    bat 'set'
                }
            }  
        }
        stage('Building') {
            steps {
                script {
                    dir("${DIR_NAME}") {
                        echo "Building..."
                        unstash "stash-${params.pname}"
                        bat "mvn clean package -DskipTests=true"
                    }
                }
            }
        }
        stage('Testing') {
            steps {
                script {
                    dir("${DIR_NAME}") {
                        echo "Testing..."
                        unstash "stash-${params.pname}"
                        bat "mvn test"
                    }
                }
            }
        }
        // stage('Dependency Check') {
        //     steps {
        //         script {
        //             dir("${DIR_NAME}") {
        //                 echo "Dependency Check..."
        //                 unstash "stash-${params.pname}"
        //                 bat "mvn org.owasp:dependency-check-maven:check"
        //             }
        //         }
        //     }
        // }
        stage('Building Docker Image') {
            when{
                expression{
                    return BRANCH =~ /(release|bugfix)-*([a-z0-9]*)/ || BRANCHNAME == 'develop'
                }
            }
            steps {
                script {
                    dir("${DIR_NAME}") {
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
               cleanWs()
            }
        }
    }
}
