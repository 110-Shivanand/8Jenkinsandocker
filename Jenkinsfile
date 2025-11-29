pipeline {
  agent any

  environment {
    IMAGE_NAME = "blinkit"
    IMAGE_TAG  = "latest"
    CONTAINER_NAME = "blinktcontainer"
    CRED_ID = "docker-hub-creds"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Info') {
      steps {
        bat '''
          echo ===== Node info =====
          ver
          echo.
          echo ===== Docker version =====
          docker --version 2>nul || echo "docker not found or not on PATH"
          echo =====================
        '''
      }
    }

    stage('Cred test') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.CRED_ID, usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
          bat '''
            echo Jenkins found Docker Hub user: %DOCKER_HUB_USER%
          '''
        }
      }
    }

    stage('Build image') {
      steps {
        bat '''
          docker build -t %IMAGE_NAME%:%IMAGE_TAG% .
        '''
      }
    }

    stage('Login & Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.CRED_ID, usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
          bat '''
            echo %DOCKER_HUB_PASS% | docker login -u %DOCKER_HUB_USER% --password-stdin
            docker tag %IMAGE_NAME%:%IMAGE_TAG% %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG%
            docker push %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG%
          '''
        }
      }
    }

    stage('Deploy (stop & run)') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.CRED_ID, usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
          bat '''
            echo %DOCKER_HUB_PASS% | docker login -u %DOCKER_HUB_USER% --password-stdin
            docker pull %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG% || exit /b 0
            docker rm -f %CONTAINER_NAME% || exit /b 0
            docker run -d -p 3000:3000 --name %CONTAINER_NAME% %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG%
          '''
        }
      }
    }
  }

  post {
    always {
      cleanWs()
    }
    success {
      bat """
        echo Deployment succeeded.
        echo Deployed %CONTAINER_NAME% from %DOCKER_HUB_USER%/%IMAGE_NAME%:%IMAGE_TAG%
      """
    }
    failure {
      echo "Build or deploy failed"
    }
  }
}
