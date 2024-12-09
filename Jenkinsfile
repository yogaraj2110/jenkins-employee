pipeline {
    agent any

    environment {
        ACCESS_TOKEN = "ABCD1234EFGHabcd1234efgh5678!@#\$%&*()"
        REFRESH_TOKEN = "abcd1234efgh5678!@#\$%&*()ABCD1234EFGH"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Run SonarScanner
                    withSonarQubeEnv('SonarQube') { // 'SonarQube' is the name you configured in Jenkins SonarQube settings
                        sh """
                        sonar-scanner \
                          -Dsonar.projectKey=ecommerce-portal \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=${env.SONAR_HOST_URL} \
                          -Dsonar.login=${env.SONAR_AUTH_TOKEN}
                        """
                    }
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
