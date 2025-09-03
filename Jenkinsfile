pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/jimagrini"   // cambia si usas otro registry
        IMAGE_NAME = "notas"
        KUBECONFIG = credentials('kubeconfig-cred') // credencial de Jenkins con kubeconfig
        DOCKER_CREDENTIALS = credentials('dockerhub-cred') // usuario y pass de dockerhub
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
                    docker build -t $REGISTRY/$IMAGE_NAME:\$BUILD_NUMBER .
                    """
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // si tienes tests con pytest u otro, agrégalo aquí
                    sh "echo 'No hay tests definidos, skipping...'"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh """
                    echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin
                    docker push $REGISTRY/$IMAGE_NAME:\$BUILD_NUMBER
                    docker logout
                    """
                }
            }
        }

        stage('Install kubectl') {
            steps {
                script {
                    sh """
                    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    mv kubectl /usr/local/bin/
                    kubectl version --client
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl --kubeconfig=\$KUBECONFIG set image deployment/notas-deployment notas=$REGISTRY/$IMAGE_NAME:\$BUILD_NUMBER --record
                    kubectl --kubeconfig=\$KUBECONFIG rollout status deployment/notas-deployment
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
