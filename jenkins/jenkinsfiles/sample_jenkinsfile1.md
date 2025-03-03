pipeline {
    agent { label 'Agent1' }

    environment {
        GITHUB_REPO_URL = 'https://github.com/florayuyuun123/Jenkins-pipeline-project.git'
        BRANCH_NAME = 'main'
        CREDENTIALS_ID = 'jenkins-github-creds'
        IMAGE_NAME = 'flo-movies-image'
        CONTAINER_NAME = 'flo-movies-new'
    }

    stages {
        stage('Clone Repository') {
            steps {
                cleanWs()
                git branch: "${env.BRANCH_NAME}", 
                    url: "${env.GITHUB_REPO_URL}", 
                    credentialsId: "${env.CREDENTIALS_ID}"
            }
        }

        stage('Check for Required Files') {
            steps {
                sh 'ls -la'
                sh 'test -f pom.xml || (echo "❌ pom.xml not found!" && exit 1)'
                sh 'test -f Dockerfile || (echo "❌ Dockerfile not found!" && exit 1)'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh 'docker ps -aq --filter "name=${CONTAINER_NAME}" | xargs -r docker stop'
                    sh 'docker ps -aq --filter "name=${CONTAINER_NAME}" | xargs -r docker rm'
                    sh 'docker run --name ${CONTAINER_NAME} --rm -d -p 8008:8080 ${IMAGE_NAME}'
                }
            }
        }
    }

    post {
        always {
            echo '🔄 Cleaning up old Docker containers and images...'
            sh 'docker ps -aq --filter "name=${CONTAINER_NAME}" | xargs -r docker stop'
            sh 'docker ps -aq --filter "name=${CONTAINER_NAME}" | xargs -r docker rm'
            sh 'docker images --filter=reference="${IMAGE_NAME}" -q | xargs -r docker rmi'
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
