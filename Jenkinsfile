pipeline {
    agent any
    tools{
        maven 'maven3'
        jdk 'jdk17'
    }
    environment {
        PROJECT_NAME = "${params.pname}"
        DIR_NAME = "run-${PROJECT_NAME}-${params.BUILD_NUMBER}"
        DEPLOYENV = 'NONE'
        BRANCH = "${params.branch}"
        VERSION_TRACKER_SERVER = 'http://localhost:1212/api/v1/versioning'
        TAG    = ""
        MAJOR_VERSION = 1
        MINOR_VERSION = 0
        PATCH_VERSION = 0
        REGISTRY_CREDENTIALS = 'docker-hub-up'
        REGISTRY_REPO_NAME = "mounirelbakkali"
    }
    stages {
        stage('Cloning Git') {
            steps {
                script {
                    echo "Cloning Git..."
                    echo "branch name: ${BRANCH}"
                    git branch: "${BRANCH}", credentialsId: 'github', url: "${params.url}"
                    stash includes: '**', name: "stash-${PROJECT_NAME}"
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
        stage('Get Current Version'){
            parallel{
                stage('DEV'){
                    when{
                        expression{
                            return BRANCH =~ /(feature)-*([a-z0-9]*)/
                        }
                    }
                    environment{
                        DEPLOYENV = 'DEV'
                    }
                    steps{
                        script{
                            DEPLOYENV = 'DEV'
                            final String response = bat(script: "curl -s ${VERSION_TRACKER_SERVER}/${PROJECT_NAME}/${DEPLOYENV}", returnStdout: true).trim()
                            TAG = response.tokenize('\n').last().trim()
                            echo "---TAG: ${TAG}"
                        }
                    }
                }
                stage('PROD'){
                    when{
                        expression{
                            return BRANCH =~ /(release|bugfix)-*([a-z0-9]*)/ || BRANCH == 'develop'
                        }
                    }
                    environment{
                        DEPLOYENV = 'PROD'
                    }
                    steps{
                        script{
                            DEPLOYENV = 'PROD'
                            final String response = bat(script: "curl -s ${VERSION_TRACKER_SERVER}/${PROJECT_NAME}/${DEPLOYENV}", returnStdout: true).trim()
                            TAG = response.tokenize('\n').last().trim()
                            echo "---TAG: ${TAG}"
                        }
                    }
                }
            }
        }
        stage('Building') {
            steps {
                script {
                    dir("${DIR_NAME}") {
                        echo "Building..."
                        unstash "stash-${PROJECT_NAME}"
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
                        unstash "stash-${PROJECT_NAME}"
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
        //                 unstash "stash-${PROJECT_NAME}"
        //                 bat "mvn org.owasp:dependency-check-maven:check"
        //             }
        //         }
        //     }
        // }
        stage('Building and Pushing Docker Image') {
            when{
                expression{
                    return BRANCH =~ /(release|bugfix)-*([a-z0-9]*)/ || BRANCH == 'develop'
                }
            }
            steps{
                    script{
                         dir("${DIR_NAME}") {
                            unstash "stash-${PROJECT_NAME}"
                            def releaseType = determineReleaseType()
                            echo "Release Type: ${releaseType}"
                            def imageName = "${REGISTRY_REPO_NAME}/${PROJECT_NAME}:${TAG}"
                            echo "Building Docker Image : ${imageName}"
                            def app = docker.build("${imageName}")
                            docker.withRegistry("", REGISTRY_CREDENTIALS) {
                                app.push()
                            }
                            // bat """git tag -a v${TAG} -m "New Release" """
                            // bat "git push origin v${TAG}"
                        }   
                        
                    }
                }  
        }
        stage("Update Version"){
            steps{
                script{
                    def releaseType = determineReleaseType()
                    bat """curl -X POST -H \"Content-Type: application/json\" -d \"{\\\"serviceName\\\":\\\"${PROJECT_NAME}\\\",\\\"deploymentEnv\\\":\\\"${DEPLOYENV}\\\",\\\"versionPart\\\":\\\"${releaseType}\\\"}\" http://localhost:1212/api/v1/versioning"""        
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

def determineReleaseType() {
        if (BRANCH.endsWith('major-release')) {
            return 'MAJOR'
        } else if (BRANCH.endsWith('release') || BRANCH == 'develop') {
            return 'MINOR'
        } else if (BRANCH =~ /(bugfix)-*([a-z0-9]*)/ ) {
            return 'PATCH'
        } else {
            return 'UNKNOWN'
        }
    }