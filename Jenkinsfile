pipeline {
  agent any
  options {
    timestamps()
    disableConcurrentBuilds()
  }
  triggers {
    githubPush()    // se dispara con el webhook
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
        // traigo las 3 variables que creaste en Jenkins
        withCredentials([
          string(credentialsId: 'SSH_HOST',        variable: 'SSH_HOST'),
          string(credentialsId: 'SSH_PORT',        variable: 'SSH_PORT'),
          string(credentialsId: 'SSH_REMOTE_USER', variable: 'SSH_REMOTE_USER')
        ]) {
          // uso la clave ssh que está en Jenkins
          sshagent(credentials: ['ssh-demo']) {
            sh '''
              set -eu pipefail

              echo "[INFO] Copiando archivos al servidor remoto..."
              # ojo: mandamos TODO, incluso .git, pero allá el deploy.sh lo ignora
              scp -P ${SSH_PORT} \
                -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no \
                -r . ${SSH_REMOTE_USER}@${SSH_HOST}:/config/src

              echo "[INFO] Ejecutando script de deploy remoto..."
              ssh -p ${SSH_PORT} \
                -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no \
                ${SSH_REMOTE_USER}@${SSH_HOST} '/config/deploy.sh'
            '''
          }
        }
      }
    }
  }

  post {
    success {
      echo '✅ Deploy finalizado. Podés ver el resultado en http://localhost:8081/ (o en el puerto que tenga tu contenedor web)'
    }
    failure {
      echo '❌ Error en el deploy. Revisá los logs del pipeline.'
    }
  }
}
