pipeline {
    agent any

    tools {
        git 'git'  // Specify the Git tool configured in Jenkins
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
                    // Start SonarQube using Docker
                    sh """
                    docker run -d -p 9000:9000 --name sonarqube sonarqube
                    """
                }
            }
        }

        stage('Wait for SonarQube to be Ready') {
            steps {
                script {
                    // Poll until SonarQube is accessible
                    def maxRetries = 30
                    def retries = 0
                    def sonarReady = false

                    while (retries < maxRetries) {
                        def response = sh(
                            script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:9000/api/system/ping",
                            returnStatus: true
                        )

                        if (response == 0) {
                            echo "SonarQube is ready!"
                            sonarReady = true
                            break
                        }

                        echo "SonarQube not ready, retrying... (${retries + 1}/${maxRetries})"
                        retries++
                        sleep 10 // Wait for 10 seconds before retrying
                    }

                    if (!sonarReady) {
                        error "SonarQube server not ready after ${maxRetries} attempts"
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarScanner with Docker
                    sh """
                    docker run --rm \
                      -v "${env.WORKSPACE}:/usr/src" \
                      sonarsource/sonar-scanner-cli \
                      -Dsonar.projectKey=ecommerce-portal \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=${env.SONAR_HOST_URL} \
                      -Dsonar.login=${env.SONAR_AUTH_TOKEN}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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

    post {
        always {
            echo "Cleaning up Docker containers..."
            sh "docker rm -f sonarqube || true"
        }
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
