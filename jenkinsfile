pipeline {
  agent any

  environment {
    IMAGE_NAME = "blinkit"
    IMAGE_TAG  = "latest"
    CONTAINER_NAME = "blinktcontainer"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build image') {
      steps {
        script {
          sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
        }
      }
    }

    stage('Login & Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
          script {
            sh '''
              echo "$DOCKER_HUB_PASS" | docker login -u "$DOCKER_HUB_USER" --password-stdin
              docker tag ${IMAGE_NAME}:${IMAGE_TAG} $DOCKER_HUB_USER/${IMAGE_NAME}:${IMAGE_TAG}
              docker push $DOCKER_HUB_USER/${IMAGE_NAME}:${IMAGE_TAG}
            '''
          }
        }
      }
    }

    stage('Deploy (stop & run)') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
          script {
            sh '''
              echo "$DOCKER_HUB_PASS" | docker login -u "$DOCKER_HUB_USER" --password-stdin

              docker pull $DOCKER_HUB_USER/${IMAGE_NAME}:${IMAGE_TAG} || true

              docker rm -f ${CONTAINER_NAME} || true

              docker run -d -p 3000:3000 --name ${CONTAINER_NAME} $DOCKER_HUB_USER/${IMAGE_NAME}:${IMAGE_TAG}
            '''
          }
        }
      }
    }
  }

  post {
    always {
      cleanWs()
    }
    success {
      echo "Deployed ${CONTAINER_NAME} from $DOCKER_HUB_USER/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    failure {
      echo "Build or deploy failed"
    }
  }
}
