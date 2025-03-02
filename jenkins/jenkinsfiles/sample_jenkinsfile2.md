# Tested and working

This sample Jenkinsfile is expected to archive the following:

- Clone your `gitHub repository url`
- Build the code with the `'mvn clean package'` goal
- Build a `docker image` from the `war file`
- Login to your `dockerHub`
- Push the docker image to dockerHub
- Then run a `docker container`

## Sample Jenkinsfile2

```python
pipeline {
    agent { label 'Agent1' }  // Replace 'wsl-agent' with the label of your agent

    environment {
        GITHUB_REPO_URL = 'https://github.com/florayuyuun123/Jenkins-pipeline-project.git'
        BRANCH_NAME = 'main'  // Replace with your branch name if it's not 'main'
        GITHUB_CREDENTIALS_ID = 'jenkins-github-creds'  // Replace with your Jenkins GitHub credentials ID
        DOCKERHUB_CREDENTIALS_ID = 'jenkins-docker-cred'  // Replace with your Jenkins Docker Hub credentials ID
        DOCKERHUB_REPO = 'yuyuun/jenkins-pipeline'  // Replace with your Docker Hub repository
    }

    stages {
        stage('Agent Details') {
            steps {
                echo "Running on agent: ${env.NODE_NAME}"
                sh 'uname -a'  // Print system information
                sh 'whoami'    // Print the current user
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: "${env.GITHUB_REPO_URL}", credentialsId: "${env.GITHUB_CREDENTIALS_ID}"
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'  // Simple Maven build
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh 'docker --version'  // Verify Docker installation
                    sh "docker build -t ${env.DOCKERHUB_REPO}:latest ."  // Build Docker image
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${env.DOCKERHUB_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        sh "docker push ${env.DOCKERHUB_REPO}:latest"
                        sh 'docker logout'
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh "docker run --name yuyuun-jenkins-pipeline-con --rm -d -p 8000:8080 ${env.DOCKERHUB_REPO}:latest"  // Run Docker container in detached mode
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Docker containers and images...'
            sh 'docker rm $(docker ps -a -q) || true'
            sh 'docker rmi $(docker images -q) || true'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

## Sample Dockerfile

```python
# Use the official Tomcat 9 image as the base image
FROM tomcat:9-jdk11-openjdk

# Remove the default web applications deployed with Tomcat
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy the WAR file into the webapps directory of Tomcat
# Replace 'your-app.war' with the name of your WAR file
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war


# Expose port 8080 for the Tomcat server
EXPOSE 8080

# Start Tomcat server
CMD ["catalina.sh", "run"]
```
