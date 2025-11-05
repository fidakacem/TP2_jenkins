pipeline {
  agent any
  options { timestamps() }

  environment {
    IMAGE = 'fidakacem/monapp' 
    TAG   = "build-${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Docker Build') {
      steps {
        
        script {
          if (isUnix()) {
            sh 'docker --version'
            sh "docker build -t ${env.IMAGE}:${env.TAG} ."
          } else {
            bat 'docker version'
            bat "docker build -t %IMAGE%:%TAG% ."
          }
        }
      }
    }

    stage('Smoke Test') {
      steps {
        script {
          if (isUnix()) {
            sh '''
              docker rm -f monapp_test || true
              docker run -d --name monapp_test -p 8081:80 ${IMAGE}:${TAG}
              sleep 3
              curl -I http://localhost:8081 | grep "200 OK"
              docker rm -f monapp_test
            '''
          } else {
            bat '''
              docker rm -f monapp_test 2>nul || ver > nul
              docker run -d --name monapp_test -p 8081:80 %IMAGE%:%TAG%
              ping -n 3 127.0.0.1 > nul
              curl -I http://localhost:8081 | find "200 OK"
              docker rm -f monapp_test
            '''
          }
        }
      }
    }

    stage('Push (Docker Hub)') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                          usernameVariable: 'DOCKER_USER',
                                          passwordVariable: 'DOCKER_PASS')]) {
          script {
            if (isUnix()) {
              sh '''
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                docker tag ${IMAGE}:${TAG} ${IMAGE}:latest
                docker push ${IMAGE}:${TAG}
                docker push ${IMAGE}:latest
              '''
            } else {
              bat '''
                echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                docker tag %IMAGE%:%TAG% %IMAGE%:latest
                docker push %IMAGE%:%TAG%
                docker push %IMAGE%:latest
              '''
            }
          }
        }
      }
    }
  }

  post {
    success { echo 'Build+Test+Push OK' }
    failure { echo 'Build/Tests/Push KO' }
  }
}
