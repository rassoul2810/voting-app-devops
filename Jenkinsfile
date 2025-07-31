pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = "docker-compose.yml"

        // Variables de l’environnement issues de .env (fixées ici directement)
        VOTE_PORT = "5000"
        RESULT_PORT = "5001"
        DB_NAME = "postgres"
        DB_USER = "postgres"
        DB_PASSWORD = "changeme"
    }

    stages {

        stage('Vérification Docker & Compose') {
            steps {
                sh 'docker --version'
                sh 'docker compose version'
            }
        }

        stage('Checkout') {
            steps {
                git url: 'https://github.com/rassoul2810/voting-app-devops.git', branch: 'main'
            }
        }

        stage('Nettoyage soft') {
            steps {
                sh '''
                echo "Suppression des conteneurs voting sauf Jenkins..."
                docker ps -a --format '{{.Names}}' | grep voting- | grep -v voting-jenkins | xargs -r docker rm -f || true
                '''
            }
        }

        stage('Build des services') {
            steps {
                sh 'docker compose -f $DOCKER_COMPOSE_FILE build --pull'
            }
        }

        stage('Démarrage des services') {
            steps {
                // On exclut Jenkins du démarrage pour éviter le conflit de nom
                sh 'docker compose -f $DOCKER_COMPOSE_FILE up -d vote result worker redis db'
            }
        }

        stage('Vérification des services') {
            steps {
                sh '''
                docker compose -f $DOCKER_COMPOSE_FILE ps
                echo "Vérif :"
                curl -s http://localhost:$VOTE_PORT || echo "Vote app non dispo"
                curl -s http://localhost:$RESULT_PORT || echo "Result app non dispo"
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline terminé.'
        }
        failure {
            echo 'Pipeline en échec.'
        }
    }
}

