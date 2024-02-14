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
        VERSION_TRACKER_SERVER = 'http://localhost:1212'
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
        stage('Get Current Version'){
            parralel{
                stage('DEV'){
                    when{
                        expression{
                            return BRANCHNAME =~ /(feature)-*([a-z0-9]*)/
                        }
                    }
                    environment{
                        DEPLOYENV = 'DEV'
                    }
                    steps{
                        script{
                            TAG = bat(script: "curl -X GET ${VERSION_TRACKER_SERVER}/${params.pname}/${DEPLOYENV}", returnStdout: true).trim()
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
                           TAG = bat(script: "curl -X GET ${VERSION_TRACKER_SERVER}/${params.pname}/${DEPLOYENV}", returnStdout: true).trim()
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
        stage('Building and Pushing Docker Image') {
            when{
                expression{
                    return BRANCH =~ /(release|bugfix)-*([a-z0-9]*)/ || BRANCH == 'develop'
                }
            }
            steps{
                    script{
                        def releaseType = determineReleaseType()
                        echo "Release Type: ${releaseType}"
                        def jsonBody = "{"serviceName": "${params.pname}"},"deploymentEnv": "${DEPLOYENV}","versionPart": "${releaseType}"}"
                        bat "curl -X POST -H 'Content-Type: application/json' -d '${jsonBody}' ${VERSION_TRACKER_SERVER}"
                        def app = docker.build("${REGISTRY_REPO_NAME}/${params.pname}:${TAG}")
                        docker.withRegistry("", REGISTRY_CREDENTIALS) {
                            app.push()
                        }
                        bat """git tag -a v${TAG} -m "New Release" """
                        bat "git push origin v${TAG}"
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
        if (env.BRANCH_NAME.endsWith('major-release') || env.BRANCH_NAME == 'develop') {
            return 'MAJOR'
        } else if (env.BRANCH_NAME.endsWith('release')) {
            return 'MINOR'
        } else if (env.BRANCH_NAME =~ /(bugfix)-*([a-z0-9]*)/ ) {
            return 'PATCH'
        } else {
            return 'UNKNOWN'
        }
    }