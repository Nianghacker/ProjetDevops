pipeline {
    agent any
    environment {
        REGISTRY = "docker.io/nianghacker"
        REGISTRY_CREDENTIALS = "dockerhub_credentials"
        GIT_REPO = "https://github.com/Nianghacker/ProjetDevops.git"
        SONARQUBE_SERVER = "SonarQubeServer"
        EMAIL_RECIPIENT = "admin@example.com"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: "${env.GIT_REPO}"
            }
        }
        stage('Build Docker Images') {
            steps {
                script {
                    def services = ['backend', 'frontend']
                    for (service in services) {
                        sh "docker build -t ${REGISTRY}/${service}:latest ${service}"
                    }
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${REGISTRY_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        def services = ['backend', 'frontend']
                        for (service in services) {
                            sh "docker push ${REGISTRY}/${service}:latest"
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker-compose down || true'
                sh 'docker-compose up -d'
            }
        }
        stage('Health Check') {
            steps {
                script {
                    def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:80", returnStdout: true).trim()
                    if (response != '200') {
                        error("Health check failed. Status code: ${response}")
                    }
                }
            }
        }
    }
    post {
        success {
            mail to: "${env.EMAIL_RECIPIENT}",
                 subject: "Pipeline Jenkins SUCCESS",
                 body: "Le pipeline s'est exécuté avec succès !"
        }
        failure {
            mail to: "${env.EMAIL_RECIPIENT}",
                 subject: "Pipeline Jenkins FAILED",
                 body: "Le pipeline a échoué. Veuillez vérifier les logs Jenkins."
        }
    }
}
