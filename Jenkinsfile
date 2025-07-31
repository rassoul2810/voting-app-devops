pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = "docker-compose.yml"
    }

    stages {

        stage('🔍 Vérification des outils') {
            steps {
                sh 'echo 🔍 Vérification de Docker et Compose...'
                sh 'docker --version'
                sh 'docker compose version'
            }
        }

        stage('🧬 Cloner le dépôt Git') {
            steps {
                git url: 'https://github.com/MohamedSadio/voting-app-project.git', branch: 'main'
            }
        }

        stage('🧹 Nettoyage SÉLECTIF Docker (pas Jenkins !)') {
            steps {
                sh '''
                    echo "🧹 Suppression des services Voting SAUF Jenkins..."
                    docker rm -f voting-db voting-app voting-result voting-worker voting-redis 2>/dev/null || true

                    echo "🧼 Suppression des volumes Voting..."
                    docker volume rm voting-db-data 2>/dev/null || true

                    echo "🔁 Nettoyage conteneurs inutilisés..."
                    docker container prune -f || true

                    echo "🔗 Création réseaux si absents..."
                    docker network create --driver bridge voting-backend || true
                    docker network create --driver bridge voting-frontend || true
                '''
            }
        }

        stage('🔧 Build des images Docker') {
            steps {
                sh 'echo 🔧 Build des images avec Docker Compose...'
                sh 'docker compose -f $DOCKER_COMPOSE_FILE build --pull || true'
            }
        }

        stage('🚀 Déploiement de l’application') {
            steps {
                sh 'echo 🚀 Lancement des services...'
                sh 'docker compose -f $DOCKER_COMPOSE_FILE up -d || true'
            }
        }

        stage('✅ Vérification des services') {
            steps {
                sh '''
                    echo "🔍 État des services Docker Compose :"
                    docker compose -f $DOCKER_COMPOSE_FILE ps || true

                    echo "🌐 Test de connectivité des applis :"
                    curl -s http://localhost:5000 || echo "⚠️ Vote app non dispo"
                    curl -s http://localhost:5001 || echo "⚠️ Result app non dispo"
                '''
            }
        }
    }

    post {
        always {
            echo '📦 Pipeline terminé (succès ou échec).'
        }
        failure {
            echo '💥 Échec du pipeline. Logs récents :'
            sh 'docker compose -f $DOCKER_COMPOSE_FILE ps || true'
        }
    }
}

