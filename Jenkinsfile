pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerHub')  
        EC2_SSH_CREDENTIALS = credentials('EC2SSH')      
        DOCKER_IMAGE = "image/tage" 
        EC2_INSTANCE_IP = "51.21.219.64"            
    }

  stages {
      
        stage('Cleanup Workspace') {
            steps {
                deleteDir() // Clean the workspace to ensure fresh clone
            }
        }
        stage('Clone Repository') {
            steps {
                echo 'Cloning the repository...'
                // Use 'main' if your repo uses it instead of 'master'
                sh 'git clone -b main image:tag'
            }
        }
        stage('Check Workspace Files') {
            steps {
                sh 'ls -la'
                sh 'ls -la chatApp'// This will list all files in the workspace to check for Dockerfile
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building the Docker image...'
                sh 'docker build -t image:latest -f chatApp/Dockerfile chatApp'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Log in to Docker Hub
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerHub') {
                        // Push the built Docker image to Docker Hub
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }

        stage('Deploy to EC2') {
        steps {
            script {
            // SSH into the EC2 instance and pull & run the Docker image
            sshagent(['EC2SSH']) {
                 sh '''
ssh -o StrictHostKeyChecking=no ubuntu@51.21.219.64 <<EOF
docker pull image/chatapp:latest
docker stop chatapp || true
docker rm chatapp || true
docker run -d -p 9000:9000 --name chatapp image/chatapp:latest
EOF
'''

            }
        }
    }
}

    }

    post {
        always {
            // Clean up workspace after the pipeline
            cleanWs()
        }
    }
}
