pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials ('dockerhub')
    RepoDockerHub = 'ferraroluc'
    NameContainer = 'flask-hello-world'
  }

  stages {
    stage('Check changes') {
      steps {
        script {
          def changedFiles = sh(script: 'git diff --name-only HEAD~1 HEAD || true', returnStdout: true).trim().split('\n').findAll { it }
          env.SKIP_BUILD = (changedFiles.size() > 0 && changedFiles.every { it.startsWith('k8s/') }).toString()
        }
      }
    }

    stage('Build') {
      when { expression { env.SKIP_BUILD != 'true' } }
      steps {
        sh "docker build -t ${env.RepoDockerHub}/${env.NameContainer}:${env.BUILD_NUMBER} ."
      }
    }

    stage('Login to Dockerhub') {
      when { expression { env.SKIP_BUILD != 'true' } }
      steps {
        sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
      }
    }

    stage('Push image to Dockerhub') {
      when { expression { env.SKIP_BUILD != 'true' } }
      steps {
        sh "docker push ${env.RepoDockerHub}/${env.NameContainer}:${env.BUILD_NUMBER}"
      }
    }

    stage('Deploy container') {
      when { expression { env.SKIP_BUILD != 'true' } }
      steps {
        sh "if [ 'docker stop ${env.NameContainer}' ] ; then docker rm -f ${env.NameContainer} && docker run -d --name ${env.NameContainer} -p 5000:5000 ${env.RepoDockerHub}/${env.NameContainer}:${env.BUILD_NUMBER} ; else docker run -d --name ${env.NameContainer} -p 5000:5000 ${env.RepoDockerHub}/${env.NameContainer}:${env.BUILD_NUMBER} ; fi"
      }
    }
  }
}
