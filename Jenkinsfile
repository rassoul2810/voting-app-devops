pipeline {
  agent any
  
  environment {
    VOTE_PORT = '5000'
    RESULT_PORT = '5001'
    DOCKER_BUILDKIT = '1'
  }

  stages {

    stage('Vérification des outils') {
      steps {
        sh '''
          echo "🔍 Vérification de Docker et Compose..."
          docker --version || { echo "❌ Docker non installé"; exit 1; }
          docker compose version || { echo "❌ Docker Compose non installé"; exit 1; }
        '''
      }
    }

    stage('Cloner le dépôt Git') {
      steps {
        git branch: 'main',
            url: 'https://github.com/MohamedSadio/voting-app-project.git'
            // Pour usage pro : ajoute credentialsId: 'github-creds-id'
      }
    }

    stage('Nettoyage de l’environnement Docker') {
      steps {
        sh '''
          echo "🧹 Nettoyage des anciens conteneurs, volumes, réseaux..."
          docker compose down --remove-orphans --volumes || true
          docker network ls -q --filter "name=voting" | xargs -r docker network rm || true
          docker network ls -q --filter "name=backend" | xargs -r docker network rm || true
          docker volume prune -f || true
        '''
      }
    }

    stage('Build des images Docker') {
      steps {
        sh '''
          echo "🔧 Build des images avec Docker Compose..."
          docker compose build --pull
        '''
      }
    }

    stage('Déploiement de l’application') {
      steps {
        sh '''
          echo "🚀 Lancement des services..."
          docker compose up -d
          echo "⏳ Pause pour initialisation des services..."
          sleep 30
        '''
      }
    }

    stage('Tests de connectivité des services') {
      steps {
        sh '''
          echo "🧪 Vérification de l’état des conteneurs..."
          RUNNING=$(docker compose ps --services --filter "status=running" | wc -l)
          if [ "$RUNNING" -lt 5 ]; then
            echo "❌ Services non démarrés correctement ($RUNNING/5 actifs)"
            docker compose ps
            exit 1
          fi

          echo "🌐 Test de l'application de vote..."
          if curl -fs http://localhost:${VOTE_PORT}; then
            echo "✅ Vote App OK"
          else
            echo "❌ Vote App KO"
            exit 1
          fi

          echo "🌐 Test de l'application de résultats..."
          if curl -fs http://localhost:${RESULT_PORT}; then
            echo "✅ Result App OK"
          else
            echo "❌ Result App KO"
            exit 1
          fi
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline terminé avec succès !"
      echo "🔗 Vote App:     http://localhost:${VOTE_PORT}"
      echo "🔗 Result App:   http://localhost:${RESULT_PORT}"
    }

    failure {
      echo "💥 Échec du pipeline. Logs récents :"
      sh '''
        docker compose ps
        docker compose logs --tail=20 || true
      '''
    }

    always {
      echo "📦 Pipeline terminé (succès ou échec)."
    }
  }
}

