pipeline {
    agent any
    
    tools {
        git 'git'  // Specify the Git tool you configured in Jenkins
    }

    environment {
        ACCESS_TOKEN = "ABCD1234EFGHabcd1234efgh5678!@#\$%&*()"
        REFRESH_TOKEN = "abcd1234efgh5678!@#\$%&*()ABCD1234EFGH"
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN = credentials('sonarqube-token')
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
                    // Check if the SonarQube container already exists
                    def sonarqubeContainer = sh(script: "docker ps -aq -f name=sonarqube", returnStdout: true).trim()
        
                    if (sonarqubeContainer) {
                        // Start the existing container if it's stopped
                        sh "docker start sonarqube || true"
                    } else {
                        // Run a new container with persistent volumes
                        sh """
                        docker run -d -p 9000:9000 --name sonarqube \
                        -v sonarqube_data:/opt/sonarqube/data \
                        -v sonarqube_logs:/opt/sonarqube/logs \
                        -v sonarqube_extensions:/opt/sonarqube/extensions \
                        sonarqube
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
