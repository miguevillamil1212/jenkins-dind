pipeline {
  agent any

  options {
    skipDefaultCheckout(true)   // evitamos el checkout automático que rompe el workspace
    timestamps()
  }

  environment {
    APP_NAME  = 'demo-web'
    HOST_PORT = '8081'
    IMAGE_TAG = "demo-web:${env.BUILD_NUMBER}"
    REPO_URL  = 'https://github.com/miguevillamil1212/jenkins-dind.git'
    BRANCH    = 'main'
  }

  stages {
    stage('Workspace limpio + Checkout') {
      steps {
        deleteDir()
        checkout([
          $class: 'GitSCM',
          branches: [[name: "*/${BRANCH}"]],
          userRemoteConfigs: [[url: "${REPO_URL}"]]
        ])
        sh '''
          echo "Commit actual:"
          git log -1 --oneline || true
          echo "Contenido del repo:"
          ls -la
        '''
      }
    }

    stage('Build image') {
      steps {
        sh '''
          echo "=== DOCKER VERSION ==="
          docker version
          echo "=== BUILD IMAGE ==="
          docker build -t ${IMAGE_TAG} ./app
        '''
      }
    }

    stage('Run container') {
      steps {
        sh '''
          set -e
          echo "=== CLEANUP OLD CONTAINERS ==="
          docker ps -aq --filter "name=^/${APP_NAME}$" | xargs -r docker rm -f

          echo "=== RUN NEW CONTAINER ==="
          docker run -d --name ${APP_NAME} -p ${HOST_PORT}:80 ${IMAGE_TAG}

          echo "=== PS ==="
          docker ps --format "table {{.Names}}\\t{{.Image}}\\t{{.Ports}}"
        '''
      }
    }

    stage('Smoke test') {
      steps {
        sh '''
          echo "=== SMOKE TEST (host.docker.internal) ==="
          for i in {1..15}; do
            if curl -fsS "http://host.docker.internal:${HOST_PORT}" >/dev/null; then
              echo "SMOKE OK en host.docker.internal:${HOST_PORT}"
              exit 0
            fi
            echo "Aún no responde (intento $i/15). Reintentando en 2s..."
            sleep 2
          done

          echo "Smoke test contra host.docker.internal falló. Probando IP del contenedor..."
          CTN_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ${APP_NAME} || true)
          if [ -n "$CTN_IP" ] && curl -fsS "http://${CTN_IP}:80" >/dev/null; then
            echo "SMOKE OK vía IP del contenedor: ${CTN_IP}:80"
            exit 0
          fi

          echo "Smoke test FAILED. Últimos logs del contenedor:"
          docker logs ${APP_NAME} | tail -n 200 || true
          exit 1
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Despliegue OK → abre: http://localhost:${HOST_PORT}"
    }
    always {
      echo "Fin del pipeline"
    }
    failure {
      script {
        // Adjunta logs del contenedor si falló (no rompe si no existe)
        sh 'docker logs ${APP_NAME} > container.log 2>/dev/null || true'
        archiveArtifacts artifacts: 'container.log', onlyIfSuccessful: false, allowEmptyArchive: true
      }
    }
  }
}
