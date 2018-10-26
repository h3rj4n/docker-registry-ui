pipeline {
  options {
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10'))
    disableConcurrentBuilds()
  }
  agent {
    docker {
      image 'arm64v8/node:alpine'
      args '-v $WORKSPACE:/srv/app -w /srv/app'
    }
  }
  environment {
    IMAGE_NAME      = "registry-ui"
    TEMP_IMAGE_NAME = "registry-ui-build_${BUILD_NUMBER}"
    TAG_VERSION     = getPackageVersion()
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
        docker.withRegistry('http://192.168.178.12:5000') {
          def customImage = docker.build('$IMAGE_NAME:${env.BUILD_ID}')

          customImage.push()
        }
      }
    }
  }
  post {
    success {
      juxtapose event: 'success'
      sh 'figlet "SUCCESS"'
    }
    failure {
      juxtapose event: 'failure'
      sh 'figlet "FAILURE"'
    }
    always {
      sh 'docker rmi  $TEMP_IMAGE_NAME'
    }
  }
}

def getPackageVersion() {
  ver = sh(script: 'docker run --rm -v $(pwd):/data $DOCKER_CI_TOOLS bash -c "cat /data/package.json|jq -r \'.version\'"', returnStdout: true)
  return ver.trim()
}
