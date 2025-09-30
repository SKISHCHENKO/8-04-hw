pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }

  environment {
    // Go из /usr/local/go
    PATH = "/usr/local/go/bin:${env.PATH}"

    // === Nexus ===
    NEXUS_URL  = "http://localhost:8081" 
    NEXUS_REPO = "raw-releases"                     

    // === Имена артефакта ===
    APP_NAME    = "8-04-hw"          // логическое имя проекта/папки в RAW
    APP_VERSION = "${env.BUILD_NUMBER}" 
    OS          = "linux"
    ARCH        = "amd64"
    OUT_DIR     = "dist"
    OUT_FILE    = "${APP_NAME}-${OS}-${ARCH}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Go: deps & test') {
      steps {
        sh '''#!/usr/bin/env bash
set -euxo pipefail
go version
go mod download
# по заданию: тесты из текущего модуля
go test . -v
'''
      }
    }

    stage('Go: build binary') {
      steps {
        sh '''#!/usr/bin/env bash
set -euxo pipefail
rm -rf "${OUT_DIR}"
mkdir -p "${OUT_DIR}"

CGO_ENABLED=0 GOOS=${OS} GOARCH=${ARCH} \
  go build -trimpath -ldflags "-s -w" -o "${OUT_DIR}/${OUT_FILE}" .

# Для контроля типа файла
file "${OUT_DIR}/${OUT_FILE}" || true
'''
      }
    }

    stage('Upload to Nexus (RAW)') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'nexus-creds',
                                          usernameVariable: 'NUSER',
                                          passwordVariable: 'NPASS')]) {
          sh '''#!/usr/bin/env bash
set -euxo pipefail
# Путь, по которому файл появится в RAW (структуру задаём сами)
ART_PATH="${APP_NAME}/${APP_VERSION}/${OUT_FILE}"

# Загрузка в RAW: PUT на /repository/<repo>/<любой/путь>
curl -f -u "${NUSER}:${NPASS}" \
  --upload-file "${OUT_DIR}/${OUT_FILE}" \
  "${NEXUS_URL}/repository/${NEXUS_REPO}/${ART_PATH}"

echo "Uploaded to: ${NEXUS_URL}/repository/${NEXUS_REPO}/${ART_PATH}"
'''
        }
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: 'dist/**', fingerprint: true
      echo 'Build & Upload OK'
    }
  }
}

