pipeline {
    agent {
        docker {
            image 'jimagrini/jenkins-docker-kubectl:latest' // tu imagen personalizada
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        REGISTRY = "docker.io/jimagrini"
        IMAGE_NAME = "notas"
        KUBECONFIG = credentials('kubeconfig-cred')
        DOCKER_CREDENTIALS = credentials('dockerhub-cred')
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
                sh """
                    echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin
                    docker push $REGISTRY/$IMAGE_NAME:${BUILD_NUMBER}
                    docker logout
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    echo "$KUBECONFIG" > kubeconfig.yaml
                    export KUBECONFIG=\$(pwd)/kubeconfig.yaml
                    kubectl set image deployment/notas-deployment notas=$REGISTRY/$IMAGE_NAME:${BUILD_NUMBER} --record
                    kubectl rollout status deployment/notas-deployment
                """
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
