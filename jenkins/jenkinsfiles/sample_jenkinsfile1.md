// Tested and working

// This sample Jenkinsfile is expected to perform the following:
// - Clone your GitHub repository
// - Build the code with 'mvn clean package'
// - Build a Docker image from the WAR file
// - Run a Docker container

## Sample Jenkinsfile1

```
pipeline {
    agent { label 'Agent1' }  // Use your Jenkins agent label

    environment {
        GITHUB_REPO_URL = 'https://github.com/florayuyuun123/Jenkins-pipeline-project.git'
        BRANCH_NAME = 'main'  // Set to your branch
        CREDENTIALS_ID = 'jenkins-github-creds'  // Your GitHub credentials in Jenkins
        IMAGE_NAME = 'flo-movies-image'  // Name for the Docker image
        CONTAINER_NAME = 'flo-movies-new'  // Name for the running container
        DOCKERFILE_DIR = 'flo-tech/github/jenkins-projct'  // Path to the directory containing Dockerfile (relative to workspace)
    }

    stages {
        stage('Clone Repository') {
            steps {
                cleanWs()  // Ensure a clean workspace
                git branch: "${env.BRANCH_NAME}", 
                    url: "${env.GITHUB_REPO_URL}", 
                    credentialsId: "${env.CREDENTIALS_ID}"
            }
        }

        stage('Check for Required Files') {
            steps {
                sh 'ls -la'  // List files to verify repo cloned correctly
                sh 'find . -name "pom.xml"'  // Ensure pom.xml exists
                sh 'find . -name "Dockerfile"'  // Ensure Dockerfile exists
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'  // Compile and package the app
            }
        }

        stage('Build Docker Image') { 
            steps {
                sh '''
                export DOCKER_BUILDKIT=1  # Enable BuildKit
                export PATH=$PATH:/usr/local/bin
                export DOCKER_HOST=unix:///var/run/docker.sock  # Ensure correct Docker daemon

                docker buildx create --name mybuilder --driver docker-container || true  # Fix: Use correct buildx creation command
                docker buildx use mybuilder
                docker buildx inspect --bootstrap

                docker buildx build --tag ${IMAGE_NAME} --file ${DOCKERFILE_DIR}/Dockerfile ${DOCKERFILE_DIR}  # Corrected path to Dockerfile
                '''
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh 'docker run --name ${CONTAINER_NAME} --rm -d -p 8008:8080 ${IMAGE_NAME}'  // Run container in detached mode
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up old Docker containers and images...'
            sh 'docker ps -aq | xargs docker rm || true'  // Remove stopped containers
            sh 'docker images -q | xargs docker rmi || true'  // Remove old images
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}

```