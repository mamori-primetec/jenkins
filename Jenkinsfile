pipeline {
  agent any
  options { timestamps(); disableConcurrentBuilds() }
  triggers { githubPush() }  // activa el pipeline en cada push

  environment {
    SSH_HOST = '172.17.0.1'          // gateway del bridge Docker
    SSH_PORT = '2222'
    SSH_REMOTE_USER = 'linuxserver.io'  // usuario real del contenedor SSH
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

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
            set -euo pipefail
            echo "[INFO] Copiando archivos al servidor remoto..."
            scp -P ${SSH_PORT} -o StrictHostKeyChecking=no -r . ${SSH_REMOTE_USER}@${SSH_HOST}:/config/src

            echo "[INFO] Ejecutando script de deploy remoto..."
            ssh -p ${SSH_PORT} -o StrictHostKeyChecking=no ${SSH_REMOTE_USER}@${SSH_HOST} '/config/deploy.sh'
          '''
        }
      }
    }
  }

  post {
    success {
      echo '✅ Deploy finalizado. Podés ver el resultado en http://localhost:8081/'
    }
    failure {
      echo '❌ Error en el deploy. Revisá los logs del pipeline.'
    }
  }
}
