# Not yet Tested

This sample Jenkinsfile is expected to archive the following:

- Clone your `gitHub repository url`
- Build the code with the `'clean package'` goal
- Scan the archive with `sonarqube`
- Backup the artifact to `nexus`
- Push the application to `tomcat`

## Sample Jenkinsfile3

```python
pipeline {
    agent any

    tools {
        maven 'Maven 3.6.3'  // Ensure Maven is available on the agent
        jdk 'JDK 11'         // Ensure JDK 11 is available on the agent
    }

    environment {
        GITHUB_REPO = 'https://github.com/your-repo/your-project.git'
        SONARQUBE_SERVER = 'SonarQube' // Name configured in Jenkins for SonarQube server
        NEXUS_URL = 'http://your-nexus-server:8081/repository/maven-releases/'
        NEXUS_CREDENTIALS_ID = 'nexus-credentials' // Credentials ID configured in Jenkins for Nexus
        TOMCAT_URL = 'http://your-tomcat-server:8080/manager/text'
        TOMCAT_CREDENTIALS_ID = 'tomcat-credentials' // Credentials ID configured in Jenkins for Tomcat
    }

    stages {
        stage('Checkout') {
            steps {
                git url: env.GITHUB_REPO
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_SCANNER_HOME = tool 'SonarQube Scanner' // Ensure SonarQube Scanner is available
            }
            steps {
                withSonarQubeEnv(env.SONARQUBE_SERVER) {
                    sh "${env.SONAR_SCANNER_HOME}/bin/sonar-scanner"
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifactId = pom.artifactId
                    def version = pom.version
                    def groupId = pom.groupId.replace('.', '/')
                    def packaging = pom.packaging

                    sh "curl -v -u ${env.NEXUS_CREDENTIALS_ID} --upload-file target/${artifactId}-${version}.${packaging} ${env.NEXUS_URL}/${groupId}/${artifactId}/${version}/${artifactId}-${version}.${packaging}"
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    def artifactId = pom.artifactId
                    def version = pom.version
                    def packaging = pom.packaging

                    withCredentials([usernamePassword(credentialsId: env.TOMCAT_CREDENTIALS_ID, passwordVariable: 'TOMCAT_PASSWORD', usernameVariable: 'TOMCAT_USERNAME')]) {
                        sh "curl --upload-file target/${artifactId}-${version}.${packaging} --user ${TOMCAT_USERNAME}:${TOMCAT_PASSWORD} ${env.TOMCAT_URL}/deploy?path=/${artifactId}&update=true"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
```
