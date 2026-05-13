pipeline {
     agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('token')
        IMAGE_BACKEND = 'cheikh9708/odc_backend'
        IMAGE_FRONTEND = 'cheikh9708/odc_frontend'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/diagnec/fileRougeJenkins.git'
            }
        }

      stage('Backend Tests') {
         steps {
          dir('backend') {
            sh '''
            ls -la

            pip install -r requirements.txt
            python manage.py test
            '''
        }
    }
}
        stage('Build Backend Image') {
            steps {
                script {
                    def backendImage = docker.build("${env.IMAGE_BACKEND}", "./backend1")
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
                script {
                    def frontendImage = docker.build("${env.IMAGE_FRONTEND}", "./frontend1")
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKER_HUB_CREDENTIALS}") {
                        docker.image("${env.IMAGE_BACKEND}").push()
                        docker.image("${env.IMAGE_FRONTEND}").push()
                    }
                }
            }
        }

        stage('Deploy (Compose)') {
            steps {
                sh 'docker-compose down || true'
                sh 'docker-compose up -d'
            }
        }
    }

    post {
        success {
            echo "✅ Build succeeded"
        }
        failure {
            echo "❌ Build failed"
        }
    }
}
