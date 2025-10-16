pipeline {
  agent { label 'docker && linux' }
  options { timestamps() }
  environment {
    IMAGE = "nucleus/testapp"
    TAG   = "build-${env.BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build image') {
      steps {
        sh '''
          set -euxo pipefail
          echo "Docker version:"
          docker version
          docker build -t ${IMAGE}:${TAG} .
          docker images | head -n 5
        '''
      }
    }
    stage('Run smoke') {
      steps {
        sh '''
          set -euxo pipefail
          docker run --rm ${IMAGE}:${TAG} /bin/sh -lc 'echo OK && uname -a'
        '''
      }
    }
  }
  post {
    always {
      sh '''
        set +e
        docker rm -f $(docker ps -aq --filter "label=job=${JOB_NAME}") 2>/dev/null || true
        docker image prune -f || true
      '''
    }
  }
}
