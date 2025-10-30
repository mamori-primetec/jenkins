pipeline {
  agent any
  options { timestamps(); disableConcurrentBuilds() }
  triggers { githubPush() }

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
        withCredentials([
          string(credentialsId: 'SSH_HOST',        variable: 'SSH_HOST'),
          string(credentialsId: 'SSH_PORT',        variable: 'SSH_PORT'),
          string(credentialsId: 'SSH_REMOTE_USER', variable: 'SSH_REMOTE_USER')
        ]) {
          sshagent(credentials: ['ssh-demo']) {
            sh '''
              set -eu pipefail
              echo "[INFO] Copiando archivos al servidor remoto (sin .git)..."

              # creamos destino por las dudas
              ssh -p ${SSH_PORT} -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no ${SSH_REMOTE_USER}@${SSH_HOST} "mkdir -p /config/src"

              # copiamos SOLO lo que hace falta
              scp -P ${SSH_PORT} \
                  -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no \
                  index.html ${SSH_REMOTE_USER}@${SSH_HOST}:/config/src/

              echo "[INFO] Ejecutando script de deploy remoto..."
              ssh -p ${SSH_PORT} -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no \
                  ${SSH_REMOTE_USER}@${SSH_HOST} '/config/deploy.sh'
            '''
          }
        }
      }
    }
  }

  post {
    success {
      echo '✅ Deploy ok'
    }
    failure {
      echo '❌ Error en el deploy'
    }
  }
}
