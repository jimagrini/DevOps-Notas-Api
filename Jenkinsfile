pipeline {
    agent {
        docker {
            image 'jimagrini/jenkins-docker-kubectl:latest'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        REGISTRY = "docker.io/jimagrini"
        IMAGE_NAME = "notas"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jimagrini/DevOps-Notas-Api.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $REGISTRY/$IMAGE_NAME:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PSW')]) {
                    sh """
                        echo $DOCKER_PSW | docker login -u $DOCKER_USER --password-stdin
                        docker push $REGISTRY/$IMAGE_NAME:${BUILD_NUMBER}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG_FILE')]) {
                    sh """
                        export KUBECONFIG=$KUBECONFIG_FILE
                        kubectl set image deployment/notas-deployment notas=$REGISTRY/$IMAGE_NAME:${BUILD_NUMBER}
                        kubectl rollout status deployment/notas-deployment
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline ejecutada correctamente ✅"
        }
        failure {
            echo "Pipeline falló 🚨"
        }
    }
}
