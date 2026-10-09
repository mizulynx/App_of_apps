pipeline {
    agent {
        label 'docker'
    }
    options {
        skipDefaultCheckout()
    }
    stages {
        stage('Get Code') {
            steps {
                checkout scm
            }
        }
    }
}
