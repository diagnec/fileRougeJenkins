pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
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
                    docker.build("${env.IMAGE_BACKEND}", "./backend")
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
                }

                script {
                    docker.build("${env.IMAGE_FRONTEND}", "./frontend")
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'token') {
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
