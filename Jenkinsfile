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
                    def envVars = readFile('.env').split('\n')
                    envVars.each {
                        def pair = it.trim().split('=')
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
                // Ne pas inclure jenkins ici
                sh 'docker compose -f $DOCKER_COMPOSE_FILE up -d vote result worker redis db'
            }
        }

        stage('Vérification des services') {
            steps {
                sh '''
                docker compose -f $DOCKER_COMPOSE_FILE ps
                echo "Vérif :"
                curl -s http://localhost:5000 || echo "Vote app non dispo"
                curl -s http://localhost:5001 || echo "Result app non dispo"
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

