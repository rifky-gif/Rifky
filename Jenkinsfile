pipeline {
  agent any

  environment {
    IMAGE_NAME = 'rakaganteng/simple-app'
    REGISTRY_CREDENTIALS = 'dockerhub-credentials'
  }

  stages {

    stage('Checkout') {
      steps {
        echo 'Checkout source code dari GitHub...'
        checkout scm
      }
    }

    stage('Build Info') {
      steps {
        bat 'echo Mulai proses build pipeline (Windows Host + Docker Only)'
        bat 'docker --version'
      }
    }

    stage('Build Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.REGISTRY_CREDENTIALS, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          bat """
            echo Login ke Docker Hub...
            docker login -u %USER% -p %PASS%

            echo Membuat image Docker dari Dockerfile...
            docker build -t ${env.IMAGE_NAME}:${env.BUILD_NUMBER} .

            echo Logout dari Docker Hub...
            docker logout
          """
        }
      }
    }

    stage('Run Unit Tests (Pytest)') {
      steps {
        echo 'Menjalankan unit test di dalam container...'
        // Jalankan pytest di container; jika gagal, hentikan pipeline
        bat """
          docker run --rm ${env.IMAGE_NAME}:${env.BUILD_NUMBER} pytest -q || exit /b 1
        """
      }
    }

    stage('Push Docker Image') {
      when {
        expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
      }
      steps {
        withCredentials([usernamePassword(credentialsId: env.REGISTRY_CREDENTIALS, usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          bat """
            echo Login ke Docker Hub untuk push...
            docker login -u %USER% -p %PASS%

            echo Push image versi build...
            docker push ${env.IMAGE_NAME}:${env.BUILD_NUMBER}

            echo Tag image sebagai latest dan push ulang...
            docker tag ${env.IMAGE_NAME}:${env.BUILD_NUMBER} ${env.IMAGE_NAME}:latest
            docker push ${env.IMAGE_NAME}:latest

            echo Logout dari Docker Hub...
            docker logout
          """
        }
      }
    }

    stage('Verify Image') {
      steps {
        bat """
          echo Menampilkan daftar image yang ada di host...
          docker images
        """
      }
    }
  }

  post {
    success {
      echo 'Pipeline sukses — image berhasil dibangun, dites, dan di-push ke Docker Hub.'
    }
    failure {
      echo 'Pipeline gagal — periksa error pada tahap sebelumnya.'
    }
    always {
      echo 'Pipeline selesai dijalankan.'
    }
  }
}
