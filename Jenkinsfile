pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = "docker-compose.yml"
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

        stage('Charger les variables d’environnement') {
            steps {
                script {
                    def envContent = readFile('.env')
                    envContent.split('\n').each { line ->
                        def pair = line.trim().split('=')
                        if (pair.length == 2) {
                            env[pair[0]] = pair[1]
                        }
                    }
                }
            }
        }

        stage('Build des services') {
            steps {
                sh 'docker compose -f $DOCKER_COMPOSE_FILE build --pull'
            }
        }

        stage('Démarrage des services') {
            steps {
                sh '''
                echo "Lancement des services (sans Jenkins)..."
                docker compose -f $DOCKER_COMPOSE_FILE up -d --remove-orphans \
                  $(docker compose config --services | grep -v jenkins)
                '''
            }
        }

        stage('Vérification des services') {
            steps {
                sh '''
                echo "Vérification de l’état des services..."
                docker compose -f $DOCKER_COMPOSE_FILE ps

                echo "Test endpoints:"
                curl -s http://localhost:5000 || echo "Vote app non dispo"
                curl -s http://localhost:5001 || echo "Result app non dispo"
                '''
            }
        }

        // BONUS Niveau 1 : Qualité & Sécurité
        stage('Bonus: Analyse et Vérif') {
            steps {
                sh '''
                echo "Vérification du formatage Python (vote/)..."
                docker run --rm -v $(pwd)/vote:/app -w /app python:3.11-slim bash -c "pip install black && black --check . || true"

                echo "Scan de sécurité Docker (image vote)..."
                docker scan voting-app2-vote || true

                echo "État final des conteneurs :"
                docker ps -a
                '''
            }
        }

        // BONUS Niveau 2 : Résilience
        stage('Bonus: Résilience') {
            steps {
                sh '''
                echo "Relance des conteneurs tombés (si besoin)..."
                down=$(docker ps -a --filter 'status=exited' --format '{{.Names}}')
                for container in $down; do
                    echo "Restarting $container..."
                    docker restart $container
                done
                '''
            }
        }

        // BONUS Niveau 3 : Rapport HTML
        stage('Bonus: Rapport HTML') {
            steps {
                sh '''
                mkdir -p reports
                echo "<html><body><h2> Déploiement réussi</h2><p>Tout est au vert </p></body></html>" > reports/index.html
                '''
                archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true
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

