pipeline {
    agent any

    stages {
        stage('Clone repository') {
            steps {
                // Cloning the Git repository
                git 'https://github.com/rajatdhakad5/pipeline.git'
            }
        }

        stage('Build Docker image') {
            steps {
                script {
                    // Building the Docker image
                    dockerImage = docker.build("rajatdhakad5/htmlp")
                }
            }
        }

        stage('Push Docker image') {
            steps {
                script {
                    // Pushing the Docker image to Docker Hub
                    docker.withRegistry('https://registry.hub.docker.com', 'fe458057-6a64-4879-9bee-75fe77615b5c') {
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                // Deploying the Docker container to the EC2 instance
                sshagent(['aa0c76c4-fa3c-4f22-9ce0-33534698a5be']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ec2-user@65.0.185.211 << EOF
                    docker pull rajatdhakad5/htmlp:latest
                    docker stop web || true
                    docker rm web || true
                    docker run -d -p 80:80 --name web rajatdhakad5/htmlp:latest
                    EOF
                    '''
                }
            }
        }
    }
}
