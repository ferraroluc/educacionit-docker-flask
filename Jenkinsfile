pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials ('dockerhub')
    RepoDockerHub = 'ferraroluc'
    NameContainer = 'flask-hello-world'
  }

  stages {
    stage('Build') {
      steps {
        sh "docker build -t ${env.RepoDockerHub}/${env.NameContainer}:${env.BUILD_NUMBER} ."
      }
    }
    
    stage('Login to Dockerhub') {
      steps {
        sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
      }
    }

    stage('Push image to Dockerhub') {
      steps {
        sh "docker push ${env.RepoDockerHub}/${env.NameContainer}:${env.BUILD_NUMBER}"
      }
    }
    
    stage('Deploy container') {
      steps {
        sh "if [ 'docker stop ${env.NameContainer}' ] ; then docker rm -f ${env.NameContainer} && docker run -d --name ${env.NameContainer} -p 5000:5000 ${env.RepoDockerHub}/${env.NameContainer}:${env.BUILD_NUMBER} ; else docker run -d --name ${env.NameContainer} -p 5000:5000 ${env.RepoDockerHub}/${env.NameContainer}:${env.BUILD_NUMBER} ; fi"
      }
    }
  }
}
