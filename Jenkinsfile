pipeline {
  agent any

  stages {

    stage('Install PHP deps') {
      steps {
        sh 'composer install'
      }
    }

    stage('App setup') {
      steps {
        sh '''
          cp .env.example .env
          php artisan key:generate
        '''
      }
    }

    stage('Build frontend') {
      steps {
        sh 'npm install && npm run build'
      }
    }

    stage('Test') {
      steps {
        sh 'php artisan test'
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          mkdir -p ~/.ssh
          chmod 700 ~/.ssh

          if ! ssh-keygen -F 178.128.93.188 > /dev/null; then
            ssh-keyscan -H 178.128.93.188 >> ~/.ssh/known_hosts
          fi
          chmod 600 ~/.ssh/known_hosts

          rsync -avz --exclude='.env' --exclude='node_modules' \
            -e "ssh -o StrictHostKeyChecking=yes" \
            ./ root@178.128.93.188:/var/www/html/i42026/
        '''
      }
    }

  }

  post {
    failure {
      sh '''
        if [ -n "${TELEGRAM_TOKEN:-}" ] && [ -n "${TELEGRAM_CHAT_ID:-}" ]; then
          curl -s -X POST \
            "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \
            -d "chat_id=${TELEGRAM_CHAT_ID}" \
            -d "text=FAILED: ${JOB_NAME} build ${BUILD_NUMBER}"
        else
          echo "Skipping Telegram notification: TELEGRAM_TOKEN or TELEGRAM_CHAT_ID is not set."
        fi
      '''
    }
  }

}
