pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/jimagrini"
        IMAGE_NAME = "notas"
        KUBECONFIG = credentials('kubeconfig-cred')        // credencial en Jenkins
        DOCKER_CREDENTIALS = credentials('dockerhub-cred') // usuario/pass dockerhub
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jimagrini/DevOps-Notas-Api.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t $REGISTRY/$IMAGE_NAME:${BUILD_NUMBER} .
                    """
                }
            }
        }

        stage('Test') {
            steps {
                sh "echo 'No hay tests definidos, skipping...'"
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh """
                        echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin
                        docker push $REGISTRY/$IMAGE_NAME:${BUILD_NUMBER}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        echo "$KUBECONFIG" > kubeconfig.yaml
                        export KUBECONFIG=\$(pwd)/kubeconfig.yaml
                        kubectl set image deployment/notas-deployment notas=$REGISTRY/$IMAGE_NAME:${BUILD_NUMBER} --record
                        kubectl rollout status deployment/notas-deployment
                    """
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed 🚨"
        }
        success {
            echo "Pipeline executed successfully ✅"
        }
    }
}
