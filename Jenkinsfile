pipeline {
    agent any

    stages {
        stage('Initialize') {
            steps {
                echo 'Initializing pipeline...'
            }
        }
        stage('Build') {
            steps {
                echo 'Building the application...'
            }
        }
        stage('Test') {
            steps {
                echo 'Running unit tests...'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying to staging environment...'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
