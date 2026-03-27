pipeline {
    agent { label 'docker' }

    environment {
        AWS_REGION        = 'us-east-1'
        AWS_ACCOUNT_ID    = '640168426521'
        ECR_FRONTEND_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/frontend-app"
        ECR_BACKEND_REPO  = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/backend-app"
        EC2_HOST          = '100.53.60.170'
        EC2_USER          = 'ec2-user'
        DEPLOY_DIR        = '/opt/fullstack-app'
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

        stage('Build Docker Images') {
            steps {
                sh "pwd"
                sh "docker build -t ${ECR_FRONTEND_REPO}:latest -t ${ECR_FRONTEND_REPO}:${BUILD_NUMBER} ./frontend"
                sh "docker build -t ${ECR_BACKEND_REPO}:latest -t ${ECR_BACKEND_REPO}:${BUILD_NUMBER} ./backend"
            }
        }

        stage('Push Images') {
            steps {
                sh """
                    aws ecr get-login-password --region us-east-1 | \
                    docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    docker push ${ECR_FRONTEND_REPO}:latest
                    docker push ${ECR_FRONTEND_REPO}:${BUILD_NUMBER}

                    docker push ${ECR_BACKEND_REPO}:latest
                    docker push ${ECR_BACKEND_REPO}:${BUILD_NUMBER}
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['vm-creds']) {
                    sh """
                        scp -o StrictHostKeyChecking=no deploy/docker-compose.yml ${EC2_USER}@${EC2_HOST}:${DEPLOY_DIR}/docker-compose.yml

                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} '
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com &&
                            export FRONTEND_IMAGE=${ECR_FRONTEND_REPO}:${BUILD_NUMBER} &&
                            export BACKEND_IMAGE=${ECR_BACKEND_REPO}:${BUILD_NUMBER} &&
                            cd ${DEPLOY_DIR} &&
                            docker compose down || true &&
                            docker compose pull &&
                            docker compose up -d
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check stage logs.'
        }
    }
}