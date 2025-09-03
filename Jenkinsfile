pipeline {
    agent {
        docker {
            image 'bitnami/kubectl:latest'  
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

        stage('Test') {
            steps {
                sh "echo 'No hay tests definidos, skipping...'"
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
                kubectl --kubeconfig=${KUBECONFIG} set image deployment/notas-deployment notas=$REGISTRY/$IMAGE_NAME:${BUILD_NUMBER} --record
                kubectl --kubeconfig=${KUBECONFIG} rollout status deployment/notas-deployment
                """
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
