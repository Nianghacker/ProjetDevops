pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/ton_dockerhub"
        REGISTRY_CREDENTIALS = "dockerhub_credentials"
        GIT_REPO = "https://github.com/ton-repo/voting-app.git"
        SONARQUBE_SERVER = "SonarQubeServer"
        EMAIL_RECIPIENT = "admin@example.com"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }
        stage('Build Images') {
            steps {
                sh 'docker-compose build'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", "${REGISTRY_CREDENTIALS}") {
                        sh '''
                        docker tag backend_app ${REGISTRY}/backend_app:latest
                        docker tag frontend_app ${REGISTRY}/frontend_app:latest
                        docker push ${REGISTRY}/backend_app:latest
                        docker push ${REGISTRY}/frontend_app:latest
                        '''
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        sh '''
                        mvn clean verify sonar:sonar                         -Dsonar.projectKey=voting-app                         -Dsonar.host.url=$SONAR_HOST_URL                         -Dsonar.login=$SONAR_AUTH_TOKEN
                        '''
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker-compose down'
                sh 'docker-compose up -d'
            }
        }
        stage('Health Check') {
            steps {
                sh 'curl -f http://localhost:80 || exit 1'
            }
        }
    }
    post {
        success {
            mail to: "${EMAIL_RECIPIENT}",
                 subject: "Pipeline Jenkins SUCCESS",
                 body: "Le pipeline s'est exécuté avec succès !"
        }
        failure {
            mail to: "${EMAIL_RECIPIENT}",
                 subject: "Pipeline Jenkins FAILED",
                 body: "Le pipeline a échoué. Veuillez vérifier les logs Jenkins."
        }
    }
}
