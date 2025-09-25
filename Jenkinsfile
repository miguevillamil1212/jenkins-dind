pipeline {
  agent any

  options {
    skipDefaultCheckout(true)   // evita "Declarative: Checkout SCM"
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
        deleteDir() // limpia todo el workspace real
        checkout([
          $class: 'GitSCM',
          branches: [[name: "*/${BRANCH}"]],
          userRemoteConfigs: [[url: "${REPO_URL}"]]
        ])
        sh 'git rev-parse --is-inside-work-tree && git log -1 --oneline || true'
        sh 'ls -la'
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
          echo "=== SMOKE TEST ==="
          sleep 2
          curl -fsS http://localhost:${HOST_PORT} >/dev/null
        '''
      }
    }
  }

  post {
    success { echo "OK → http://localhost:${HOST_PORT}" }
    always  { echo "Fin del pipeline" }
  }
}
