pipeline {
  agent {
    label 'worker'
  }
   
  stages {
    stage('Git Checkout') {
      steps {
        checkout([$class: 'GitSCM',
                  branches: [[name: '*/main']],
                  userRemoteConfigs: [[url: 'https://github.com/meghnanookala/Devops_C6_Jenkins.git']]])
      }
    }
    
     stage('Stopping the container') {
      steps {
        script {
          sh 'docker kill $(docker ps -q)'
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        script {     
          sh '''
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 009172737512.dkr.ecr.us-east-1.amazonaws.com
          docker build -t c42project-${BUILD_NUMBER} .
          docker tag c42project-meghna:${BUILD_NUMBER} 009172737512.dkr.ecr.us-east-1.amazonaws.com/c42project-meghna:${BUILD_NUMBER}
          docker push 009172737512.dkr.ecr.us-east-1.amazonaws.com/c42project-meghna:${BUILD_NUMBER}
          '''
        }
      }
    }

    stage('Cleanup the docker image') {
      steps {
        script {
          sh 'docker rmi 009172737512.dkr.ecr.us-east-1.amazonaws.com/c42project-meghna:${BUILD_NUMBER}'
          sh 'docker rmi c42project-meghna:${BUILD_NUMBER}'
        }
      }
    }

    stage('Deploy the application') {
      steps {
        script {
          sh '''
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 009172737512.dkr.ecr.us-east-1.amazonaws.com
          docker pull 009172737512.dkr.ecr.us-east-1.amazonaws.com/c42project-meghna:${BUILD_NUMBER}
          docker run -d -p 8080:8081 009172737512.dkr.ecr.us-east-1.amazonaws.com/c42project-meghna:${BUILD_NUMBER}
          '''
        }
      }
    }
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
    timeout(time: 1, unit: 'HOURS')
  }
}
