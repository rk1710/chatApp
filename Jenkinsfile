// Pipeline set up
pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerHub')  
        EC2_SSH_CREDENTIALS = credentials('EC2SSH')      
        DOCKER_IMAGE = "taharamakda/chatapp" 
        EC2_INSTANCE_IP = "your-instance-ip"            
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
                sh 'git clone -b main https://github.com/TahaRamkda/chatApp.git'
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
                sh 'docker build -t taharamakda/chatapp:latest -f chatApp/Dockerfile chatApp'
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
ssh -o StrictHostKeyChecking=no ubuntu@your-instance-ip <<EOF
docker pull taharamakda/chatapp:latest
docker stop chatapp || true
docker rm chatapp || true
docker run -d -p 9000:9000 --name chatapp taharamakda/chatapp:latest
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
