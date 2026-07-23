pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'aadarshsorab/flask-devops-app'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Create Virtual Environment') {
            steps {
                echo 'Creating Python virtual environment...'
                sh 'python3 -m venv jenkins-venv'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Python dependencies...'
                sh './jenkins-venv/bin/pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running automated tests...'
                sh './jenkins-venv/bin/pytest'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t $DOCKER_IMAGE:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    docker push $DOCKER_IMAGE:latest
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to Kubernetes...'

                sh '''
                kubectl apply -f deployment.yaml
                kubectl rollout restart deployment flask-devops-app
                kubectl rollout status deployment flask-devops-app
                '''
            }
        }

        stage('Run Docker Container') {
            steps {
                echo 'Running Docker container...'

                sh '''
                docker stop flask-app || true
                docker rm flask-app || true
                docker run -d -p 5000:5000 --name flask-app $DOCKER_IMAGE:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}
