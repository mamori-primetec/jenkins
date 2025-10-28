pipeline {
  agent any
  options { timestamps(); disableConcurrentBuilds() }
  triggers { githubPush() }  // usa Webhook (ver paso 4)
  environment {
    SSH_HOST = '127.0.0.1'
    SSH_PORT = '2222'
    SSH_USER = 'abc' // demo
  }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Stamp') {
      steps {
        sh '''
          COMMIT=$(git rev-parse --short HEAD)
          NOW=$(date '+%Y-%m-%d %H:%M:%S')
          sed -i "s/__COMMIT__/${COMMIT}/" index.html || true
          sed -i "s/__TIME__/${NOW}/" index.html || true
        '''
      }
    }
    stage('Deploy via SSH') {
      steps {
        sh '''
          set -eu pipefail
          scp -P ${SSH_PORT} -o StrictHostKeyChecking=no -r . ${SSH_USER}@${SSH_HOST}:/root/src
          ssh -p ${SSH_PORT} -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} '/root/deploy.sh'
        '''
      }
    }
  }
  post {
    success {
      echo 'Abrí http://localhost:8081/ y /last_deploy.txt'
    }
  }
}
