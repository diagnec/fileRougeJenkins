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
            python3 -m venv venv

            . venv/bin/activate

            pip install -r requirements.txt

            python manage.py test
            '''
        }
    }
}
        stage('Build Backend Image') {
            steps {
                script {
                    def backendImage = docker.build("${env.IMAGE_BACKEND}", "./backend")
                }
            }
        }

      stage('Build Frontend Image') {
    steps {
        dir('frontend') {
            sh '''
            npm install
            npm run build
            '''

            sh 'docker build -t cheikh9708/odc_frontend .'
        }
    }
}
        stage('Push Docker Images') {
       docker.withRegistry('https://index.docker.io/v1/', 'token') {
    sh "docker push ${env.IMAGE_BACKEND}"
    sh "docker push ${env.IMAGE_FRONTEND}"
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
