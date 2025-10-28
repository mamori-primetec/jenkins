pipeline {
  agent any
  options { timestamps(); disableConcurrentBuilds() }
  // Para salir del paso sin webhook, podés usar Poll SCM 1/min. Para “instantáneo”, configurá el webhook y cambiá a githubPush().
  triggers { pollSCM('* * * * *') }
  environment {
    SSH_HOST = 'zb6.nucleus.ar'   // IP del host donde corren los containers
    SSH_PORT = '2222'
    SSH_USER = 'root'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Marcas de build') {
      steps {
        sh '''
          COMMIT=$(git rev-parse --short HEAD)
          NOW=$(date '+%Y-%m-%d %H:%M:%S')
          sed -i "s/__COMMIT__/${COMMIT}/" index.html || true
          sed -i "s/__TIME__/${NOW}/" index.html || true
        '''
      }
    }
    stage('Deploy via SSH (demo)') {
      steps {
        // Usamos scp/ssh con password para la demo rápida.
        // El contenedor de Jenkins suele traer ssh/scp. Si no, decime y te paso variante con credenciales SSH del plugin.
        sh '''
          set -euo pipefail
          # Subimos los archivos al "remoto"
          scp -P ${SSH_PORT} -o StrictHostKeyChecking=no -r . ${SSH_USER}@${SSH_HOST}:/root/src
          # Ejecutamos el script de deploy que mueve /root/src -> /var/www/html
          ssh -p ${SSH_PORT} -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} '/root/deploy.sh'
        '''
      }
    }
  }
  post {
    success {
      echo 'Abrí: http://zb6.nucleus.ar:8081  (verás commit y hora).'
      echo 'También: http://zb6.nucleus.ar:8081/last_deploy.txt'
    }
  }
}
