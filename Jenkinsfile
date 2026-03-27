pipeline {
    agent { label 'docker' }
    triggers {
        pollSCM('H/2 * * * *')   // every 2 minutes
    }
    // environment {
    //     REGISTRY = '640168426521.dkr.ecr.us-east-1.amazonaws.com'  // e.g., '123456789.dkr.ecr.us-east-1.amazonaws.com' for ECR
    //     // IMAGE_NAME_FRONT = 'frontend-app'
    //     // IMAGE_NAME_BACK = 'backend-app'
    //     // TAG = "${BUILD_NUMBER}"
    //     // FULL_IMAGE_NAME = "${REGISTRY}/${IMAGE_NAME}:${TAG}"
    // }
    environment {
    ECR_FRONTEND_REPO = '640168426521.dkr.ecr.us-east-1.amazonaws.com/frontend-app'
    ECR_BACKEND_REPO  = '640168426521.dkr.ecr.us-east-1.amazonaws.com/backend-app'
    }
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
            sh "docker --version"
            sh "docker ps -a"
            sh "docker-compose --version"
            }
        }

        //  stage('Clean up old images and containers') {
        //     steps {
        //         sh "docker-compose down"
        //         sh "docker image prune -f"
        //         sh "docker container prune -f"
        //         sh "docker image rmi -f ${FULL_IMAGE_NAME} || true"
        //     }
        // } 
        stage('Build Docker Images') {
            steps {
                sh """
                    docker build -t ${ECR_FRONTEND_REPO}:latest -t ${ECR_FRONTEND_REPO}:${_NUMBER} ./frontend
                    docker build -t ${ECR_BACKEND_REPO}:latest -t ${ECR_BACKEND_REPO}:${BUILD_NUMBER} ./backend
                """
            }
        }
        stage('Push Images') {
            steps {
                sh 'docker push ${REGISTRY}/frontend-app:v1'
                sh 'docker push ${REGISTRY}/backend-app:v1'
            }
        }
        stage('Deploy to EC2') {
            steps {
                sh '''
                ssh ec2-user@<ec2-ip> "
                docker pull ${REGISTRY}/frontend-app:v1 &&
                docker pull ${REGISTRY}/backend-app:v1 &&
                docker-compose up -d
                "
                '''
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
