pipeline {
  agent any

  environment {
    APP_NAME  = 'demo-web'
    HOST_PORT = '8081'
    IMAGE_TAG = "demo-web:${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/miguevillamil1212/jenkins-php2.git'
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
        '''
      }
    }

    stage('Smoke test') {
      steps {
        sh '''
          sleep 2
          curl -fsS http://localhost:${HOST_PORT} || (echo "Smoke test failed" && exit 1)
        '''
      }
    }
  }

  post {
    success {
      echo "Aplicación disponible en: http://localhost:${HOST_PORT}"
    }
    failure {
      echo "Pipeline falló, revisa logs"
    }
  }
}
