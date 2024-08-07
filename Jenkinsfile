pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/rajatdhakad5/pipeline.git'
        GIT_BRANCH = 'main'
        DOCKER_IMAGE_NAME = 'rajatdhakad5/htmlp'
        DOCKER_REGISTRY = 'https://registry.hub.docker.com'
        DOCKER_CREDENTIALS_ID = 'fe458057-6a64-4879-9bee-75fe77615b5c'
        SSH_CREDENTIALS_ID = 'aa0c76c4-fa3c-4f22-9ce0-33534698a5be'
        EC2_USER = 'ec2-user'
        EC2_HOST = '65.0.185.211'
    }

    stages {
        stage('Clone repository') {
            steps {
                script {
                    try {
                        // Cloning the Git repository
                        git url: env.GIT_URL, branch: env.GIT_BRANCH
                    } catch (Exception e) {
                        error "Couldn't clone the repository. Verify the repository URL, branch, and credentials."
                    }
                }
            }
        }

        stage('Build Docker image') {
            steps {
                script {
                    try {
                        // Building the Docker image
                        dockerImage = docker.build(env.DOCKER_IMAGE_NAME)
                    } catch (Exception e) {
                        error "Failed to build Docker image. Check Dockerfile and build context."
                    }
                }
            }
        }

        stage('Push Docker image') {
            steps {
                script {
                    try {
                        // Pushing the Docker image to Docker Hub
                        docker.withRegistry(env.DOCKER_REGISTRY, env.DOCKER_CREDENTIALS_ID) {
                            dockerImage.push('latest')
                        }
                    } catch (Exception e) {
                        error "Failed to push Docker image. Verify Docker Hub credentials and repository permissions."
                    }
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                script {
                    try {
                        // Deploying the Docker container to the EC2 instance
                        sshagent([env.SSH_CREDENTIALS_ID]) {
                            sh '''
                            ssh -o StrictHostKeyChecking=no ${env.EC2_USER}@${env.EC2_HOST} << EOF
                            docker pull ${env.DOCKER_IMAGE_NAME}:latest
                            docker stop web || true
                            docker rm web || true
                            docker run -d -p 80:80 --name web ${env.DOCKER_IMAGE_NAME}:latest
                            EOF
                            '''
                        }
                    } catch (Exception e) {
                        error "Deployment to AWS EC2 failed. Verify SSH credentials and EC2 instance status."
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed.'
        }
        success {
            echo 'Pipeline succeeded.'
        }
    }
}
