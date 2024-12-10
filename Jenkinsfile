pipeline {
    agent any
    
    tools {
        git 'git'  // Specify the Git tool you configured in Jenkins
    }

    environment {
        ACCESS_TOKEN = "ABCD1234EFGHabcd1234efgh5678!@#\$%&*()"
        REFRESH_TOKEN = "abcd1234efgh5678!@#\$%&*()ABCD1234EFGH"
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN = credentials('sonarqube-token') // Use Jenkins credentials plugin
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
                        // Stop and remove the existing container to avoid conflicts
                        sh """
                        docker stop $sonarqubeContainer || true
                        docker rm $sonarqubeContainer || true
                        """
                    }
                    // Run a new SonarQube container
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
                    // Running Docker Compose with the environment variables
                    sh """
                    docker-compose -f docker-compose.yml up -d
                    """
                }
            }
        }

        // stage('SonarQube Analysis') {
        //     steps {
        //         script {
        //             withSonarQubeEnv('SonarQube') {
        //                 sh """
        //                 ./sonar-scanner/bin/sonar-scanner \
        //                 -Dsonar.projectKey=your_project_key \
        //                 -Dsonar.sources=. \
        //                 -Dsonar.host.url=$SONAR_HOST_URL \
        //                 -Dsonar.login=${env.SONAR_AUTH_TOKEN}
        //                 """
        //             }
        //         }
        //     }
        // }
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
