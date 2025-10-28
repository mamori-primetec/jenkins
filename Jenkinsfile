pipeline {
  agent any
  options { timestamps(); disableConcurrentBuilds() }
  triggers { githubPush() }  // webhook de GitHub

  environment {
    // Si estás en Docker Desktop (Win/Mac):
    // SSH_HOST = 'host.docker.internal'

    // Si estás en Linux, probá primero con 172.17.0.1 (gateway del bridge):
    SSH_HOST = 'host.docker.internal'  // cámbialo a 172.17.0.1 si hace falta
    SSH_PORT = '2222'
    SSH_USER = 'abc'                   // linuxserver/openssh-server
  }

  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Sellos') {
      steps {
        sh '''
          set -e
          COMMIT=$(git rev-parse --short HEAD)
          NOW=$(date '+%Y-%m-%d %H:%M:%S')
          sed -i "s|__COMMIT__|${COMMIT}|" index.html || true
          sed -i "s|__TIME__|${NOW}|" index.html || true
        '''
      }
    }

    stage('Deploy via SSH') {
      steps {
        sshagent(credentials: ['ssh-demo']) {
          sh '''
            set -eu pipefail
            # Subo a /config/src (home de abc en esta imagen)
            scp -P ${SSH_PORT} -o StrictHostKeyChecking=no -r . ${SSH_USER}@${SSH_HOST}:/config/src
            # Ejecuta el deploy que copia a /var/www/html (lo sirve Nginx:8081)
            ssh -p ${SSH_PORT} -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} '/config/deploy.sh'
          '''
        }
      }
    }
  }

  post {
    success {
      echo 'Listo. Abrí: http://localhost:8081/  (y /last_deploy.txt)'
    }
  }
}
