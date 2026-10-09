def frontendImage = "frontend"
def backendImage = "backend"
def frontendDockerTag = ""
def backendDockerTag = ""

pipeline {
    agent {
        label 'docker'
    }
    options {
        skipDefaultCheckout()
    }
    parameters {
        string(name: 'backendDockerTag', defaultValue: '', description: 'Backend docker image tag')
        string(name: 'frontendDockerTag', defaultValue: '', description: 'Frontend docker image tag')
    }
    stages {
        stage('Get Code') {
            steps {
                checkout scm
            }
        }
        stage('Adjust version') {
            steps {
                script {
                    backendDockerTag = params.backendDockerTag.isEmpty() ? "latest" : params.backendDockerTag
                    frontendDockerTag = params.frontendDockerTag.isEmpty() ? "latest" : params.frontendDockerTag
                    currentBuild.description = "Backend: ${backendDockerTag}, Frontend: ${frontendDockerTag}"
                }
            }
        }
        stage('Clean running containers') {
            steps {
                sh "docker rm -f frontend backend"
            }
        }
        stage('Deploy application') {
            steps {
                withEnv(["FRONTEND_IMAGE=${frontendImage}:${frontendDockerTag}", "BACKEND_IMAGE=${backendImage}:${backendDockerTag}"]) {
                    sh "docker-compose up -d"
                }
            }
        }
    }
    post {
        always {
            sh "docker-compose down"
            cleanWs()
        }
    }
}
