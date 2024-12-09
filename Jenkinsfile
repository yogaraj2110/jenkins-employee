pipeline {
    agent any
    
    tools {
        git 'git'  // Specify the Git tool you configured in Jenkins
    }

    environment {
        ACCESS_TOKEN = "ABCD1234EFGHabcd1234efgh5678!@#\$%&*()"
        REFRESH_TOKEN = "abcd1234efgh5678!@#\$%&*()ABCD1234EFGH"
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN = "85f3ca840bb5b2e78a5f206424889f9bebf7cbbd"
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

        stage('Wait for SonarQube to be Ready') {
            steps {
                script {
                    // Wait until SonarQube is accessible
                    def maxRetries = 30
                    def retries = 0
                    while (retries < maxRetries) {
                        try {
                            sh "curl -s http://localhost:9000/api/system/ping"
                            break // If the ping is successful, exit the loop
                        } catch (Exception e) {
                            retries++
                            if (retries == maxRetries) {
                                error "SonarQube server not ready after ${maxRetries} attempts"
                            }
                            sleep 10 // Wait for 10 seconds before retrying
                        }
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarScanner using Docker
                    sh """
                    docker run --rm \
                      -v "/var/lib/jenkins/workspace/employee management:/usr/src" \
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
