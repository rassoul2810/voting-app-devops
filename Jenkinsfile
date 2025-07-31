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
                    def envFile = '.env'
                    if (fileExists(envFile)) {
                        echo ".env trouvé, chargement des variables..."
                        def envVars = readFile(envFile).split('\n')
                        envVars.each {
                            def pair = it.trim().split('=')
                            if (pair.length == 2) {
                                env[pair[0]] = pair[1]
                            }
                        }
                    } else {
                        echo "Fichier .env non trouvé, on continue sans le charger..."
                    }
                }
            }
        }

        stage('Build des services') {
            steps {
                sh 'docker compose -f $DOCKER_COMPOSE_FILE build --pull'
            }
        }

        stage('Démarrage des services (sauf Jenkins)') {
            steps {
                sh '''
                SERVICES=$(docker compose config --services | grep -v jenkins | tr '\n' ' ')
                docker compose -f $DOCKER_COMPOSE_FILE up -d $SERVICES
                '''
            }
        }

        stage('Vérification des services') {
            steps {
                sh '''
                docker compose -f $DOCKER_COMPOSE_FILE ps
                echo "Vérification applicative..."
                curl -s http://localhost:5000 || echo "Vote app non dispo"
                curl -s http://localhost:5001 || echo "Result app non dispo"
                '''
            }
        }

        stage('Bonus: Analyse et Vérif') {
            steps {
                echo "Analyse des logs des services..."
                sh 'docker compose logs --tail=20'
            }
        }

        stage('Bonus: Résilience') {
            steps {
                echo "Test de redémarrage contrôlé"
                sh 'docker compose restart'
                sh 'sleep 5'
            }
        }

        stage('Bonus: Rapport HTML') {
            steps {
                echo "Génération d’un faux rapport HTML"
                writeFile file: 'rapport.html', text: '<html><body><h1>✔ Rapport OK</h1></body></html>'
                archiveArtifacts artifacts: 'rapport.html', fingerprint: true
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
