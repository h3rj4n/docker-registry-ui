pipeline {
  options {
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10'))
    disableConcurrentBuilds()
  }
  agent {
    docker {
      image 'arm64v8/node:alpine'
      args '-v $WORKSPACE:/srv/app -w /srv/app -u 0'
    }
  }
  environment {
    IMAGE_NAME      = "registry-ui"
    TEMP_IMAGE_NAME = "registry-ui-build_${BUILD_NUMBER}"
  }
  stages {
    stage ('Prepare') {
      steps {
        sh 'apk add --update --no-cache git ncurses'
      }
    }
    stage('Build') {
      steps {
        sh 'npm install'
        // First run always fails?
        sh 'yarn install'

        sh 'npm run-script build'
        sh 'rm -rf node_modules'
        sh 'yarn install --prod'
      }
    }
    stage ('Publish') {
      when {
        branch 'master'
      }
      steps {
        script {
          docker.withRegistry('http://192.168.178.12:5000') {
            def customImage = docker.build('$IMAGE_NAME:${env.BUILD_ID}')

            customImage.push()
          }
        }
      }
    }
  }
}
