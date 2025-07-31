pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
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
                git 'https://github.com/rassoul2810/voting-app-devops.git'
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
                        def envContent = readFile('.env').split('\n')
                        for (line in envContent) {
                            if (line.trim() && line.contains('=')) {
                                def (key, value) = line.tokenize('=')
                                env."${key.trim()}" = value.trim()
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
                sh "docker compose -f $DOCKER_COMPOSE_FILE build --pull"
            }
        }

        stage('Démarrage des services (sauf Jenkins)') {
            steps {
                script {
                    def services = sh(script: "docker compose config --services | grep -v jenkins | tr '\\n' ' '", returnStdout: true).trim()
                    sh "docker compose -f $DOCKER_COMPOSE_FILE up -d ${services}"
                }
            }
        }

        stage('Vérification des services') {
            steps {
                sh '''
                echo "Vérification des services..."
                docker ps
                '''
            }
        }

        // BONUS 1 - Vérif CURL
        stage('Bonus: Analyse et Vérif') {
            steps {
                sh '''
                echo "Vérification avec curl..."
                curl -s --fail http://localhost:$VOTE_PORT || echo "Vote app KO"
                curl -s --fail http://localhost:$RESULT_PORT || echo "Result app KO"
                '''
            }
        }

        // BONUS 2 - Résilience Test
        stage('Bonus: Résilience') {
            steps {
                sh '''
                echo "Redémarrage test du vote-app..."
                docker restart voting-app
                sleep 3
                docker ps | grep voting-app
                '''
            }
        }

        // BONUS 3 - Rapport HTML
        stage('Bonus: Rapport HTML') {
            steps {
                sh '''
                mkdir -p build-report
                echo "<html><head><title>Rapport Build</title></head><body>" > build-report/index.html
                echo "<h1>Rapport de Build Voting App</h1>" >> build-report/index.html
                echo "<p>Date: $(date)</p>" >> build-report/index.html
                echo "<p>Commit: $(git log -1 --pretty=format:'%h - %s (%ci)')</p>" >> build-report/index.html
                echo "<pre>" >> build-report/index.html
                docker compose ps >> build-report/index.html
                echo "</pre>" >> build-report/index.html
                echo "</body></html>" >> build-report/index.html
                '''
                archiveArtifacts artifacts: 'build-report/**', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminé avec succès.'
        }
        failure {
            echo 'Pipeline échoué.'
        }
    }
}
