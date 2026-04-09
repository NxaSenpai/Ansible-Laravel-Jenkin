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
          rsync -avz --exclude='.env' --exclude='node_modules' \
            ./ root@178.128.93.188:/var/www/html/i42026/
        '''
      }
    }

  }

  post {
    failure {
      sh '''
        curl -s -X POST \
          "https://api.telegram.org/bot${TELEGRAM_TOKEN}/sendMessage" \
          -d "chat_id=${TELEGRAM_CHAT_ID}" \
          -d "text=FAILED: ${JOB_NAME} build ${BUILD_NUMBER}"
      '''
    }
  }

}
