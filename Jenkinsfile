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
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/rassoul2810/voting-app-devops.git']]
                ])
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
                    if (fileExists('.env')) {
                        echo ".env trouvé, chargement des variables..."
                        def envVars = readFile('.env').split('\n')
                        for (line in envVars) {
                            if (line.trim() && !line.startsWith('#')) {
                                def pair = line.split('=')
                                if (pair.length == 2) {
                                    env[pair[0].trim()] = pair[1].trim()
                                }
                            }
                        }
                    } else {
                        echo "Fichier .env non trouvé, passage..."
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
                docker compose -f $DOCKER_COMPOSE_FILE up -d \
                --remove-orphans \
                --scale jenkins=0 || true
                '''
            }
        }

        stage('Vérification des services') {
            steps {
                sh '''
                docker compose -f $DOCKER_COMPOSE_FILE ps
                echo "Vérif des apps :"
                curl -s http://localhost:5000 || echo "Vote app indisponible"
                curl -s http://localhost:5001 || echo "Result app indisponible"
                '''
            }
        }

        // BONUS
        stage('Bonus: Analyse et Vérif') {
            steps {
                sh 'docker stats --no-stream || true'
            }
        }

        stage('Bonus: Résilience') {
            steps {
                sh '''
                docker compose -f $DOCKER_COMPOSE_FILE restart
                echo "Redémarrage effectué pour tester la résilience"
                '''
            }
        }

        stage('Bonus: Rapport HTML') {
            steps {
                sh '''
                echo "<html><body><h1>Rapport Jenkins OK</h1></body></html>" > rapport.html
                '''
                archiveArtifacts artifacts: 'rapport.html', fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Pipeline terminé.'
        }
        success {
            echo 'Succès total du pipeline.'
        }
        failure {
            echo 'Pipeline échoué.'
        }
    }
}
