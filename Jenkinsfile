pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }

  // Go установлен из архива в /usr/local/go — добавим в PATH для агента
  environment {
    PATH = "/usr/local/go/bin:${env.PATH}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Go: deps & test') {
      steps {
        // Включаем bash, чтобы использовать pipefail при необходимости
        sh '''#!/usr/bin/env bash
set -euxo pipefail

go version
go mod download

# как требует задание:
go test . -v
'''
      }
    }

    stage('Docker: build image') {
      when { expression { fileExists('Dockerfile') } }
      steps {
        sh '''#!/usr/bin/env bash
set -euxo pipefail

docker build -t 8-04-hw:${BUILD_NUMBER} .
'''
      }
    }
  }

  post {
    success {
      echo 'Build OK'
    }
  }
}

