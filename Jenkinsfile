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
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
        // Service names on the shared infrastructure_default network
        APPLICATION_URL = 'http://frontend:5000/'
        SELENIUM_URL = 'http://selenium_hub:4444/wd/hub'
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
        stage('Selenium tests') {
            steps {
                sh "pip3 install -r test/selenium/requirements.txt"
                // up -d returns before Flask listens; wait so the first test doesn't hit a refused connection
                sh 'timeout 30 sh -c "until curl -sf $APPLICATION_URL > /dev/null; do sleep 1; done"'
                sh "python3 -m pytest test/selenium/frontendTest.py"
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
