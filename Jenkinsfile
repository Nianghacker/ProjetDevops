pipeline {
    agent any

    environment {
        PROJECT_DIR = "voting-app"
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git url: 'https://github.com/TON_USER/TON_REPO.git', branch: 'main'
            }
        }

        stage('Construction des images Docker') {
            steps {
                dir("${env.PROJECT_DIR}") {
                    sh 'docker-compose build'
                }
            }
        }

        stage('Déploiement des microservices') {
            steps {
                dir("${env.PROJECT_DIR}") {
                    sh 'docker-compose up -d'
                }
            }
        }
    }

    post {
        success {
            echo 'Déploiement réussi !'
        }
        failure {
            echo 'Le pipeline a échoué.'
        }
    }
}
