pipeline {
    agent any

    environment {
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
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=employee-project \
                        -Dsonar.sources=. \
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
