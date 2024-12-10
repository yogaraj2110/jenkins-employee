pipeline {
    agent any
    
    tools {
        git 'git'
        sonarScanner 'SonarQube Scanner'
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
                    def sonarqubeContainer = sh(script: "docker ps -aq -f name=sonarqube", returnStdout: true).trim()
                    if (sonarqubeContainer) {
                        echo "SonarQube container already exists: $sonarqubeContainer"
                        sh """
                        docker stop $sonarqubeContainer || true
                        docker rm $sonarqubeContainer || true
                        """
                    }
                    sh """
                    docker run -d --name sonarqube \
                    -p 9000:9000 \
                    -v sonarqube_data:/opt/sonarqube/data \
                    -v sonarqube_logs:/opt/sonarqube/logs \
                    -v sonarqube_extensions:/opt/sonarqube/extensions \
                    sonarqube:latest
                    """
                }
            }
        }
        stage('Build and Run Containers') {
            steps {
                script {
                    sh """
                    docker-compose -f docker-compose.yml up -d
                    """
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                        docker run --rm \
                        -e SONAR_HOST_URL=$SONAR_HOST_URL \
                        -e SONAR_AUTH_TOKEN=$SONAR_AUTH_TOKEN \
                        -v $(pwd):/usr/src \
                        sonarsource/sonar-scanner-cli \
                        -Dsonar.projectKey=employee-project \
                        -Dsonar.sources=/usr/src \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline execution complete."
        }
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
