pipeline {
    agent { label 'docker' }
    triggers {
        pollSCM('H/2 * * * *')   // every 2 minutes
    }

    stages {
        stage('Clone Repository') {
            steps {
                    git branch: 'main',
                    url: 'git@github.com:stalinkar/fullstack-cicd-ecr-ec2-jenkins.git',
                    credentialsId: 'github-creds'
            }
        }
        stage('Check the docker cli') {
            steps {
                sh "sudo docker --version"
                sh "sudo docker ps -a"
                sh "docker-compose --version"
            }
        }
        stage('Docker build') {
            steps {
                echo "Docker build running"
                sh "sudo docker build . -t pythonapp:v1"
            }
        }

        stage('Docker Container Cerateion') {
            steps {
                echo "Running docker-compose here..."
                sh "docker-compose up -d"
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
