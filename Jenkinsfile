pipeline {
    agent any

    environment {
        ACCESS_TOKEN = "ABCD1234EFGHabcd1234efgh5678!@#\$%&*()"
        REFRESH_TOKEN = "abcd1234efgh5678!@#\$%&*()ABCD1234EFGH"
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN = credentials('sonarqube-token-id')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Start SonarQube Server') {
            steps {
                script {
                    // Start SonarQube in Docker
                    sh """
                    docker run -d -p 9000:9000 --name sonarqube sonarqube
                    """
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarScanner using Docker
                    sh """
                    docker run --rm \
                      -v \$(pwd):/usr/src \
                      sonarsource/sonar-scanner-cli \
                      -Dsonar.projectKey=ecommerce-portal \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=${env.SONAR_HOST_URL} \
                      -Dsonar.login=${env.SONAR_AUTH_TOKEN}
                    """
                }
            }
        }

        stage('Build and Run Containers') {
            steps {
                script {
                    // Running Docker Compose with the environment variables
                    sh """
                    docker-compose -f docker-compose.yml up -d
                    """
                }
            }
        }
    }
}
