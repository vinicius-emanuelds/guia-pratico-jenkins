pipeline {
    agent any

    stages {
        stage('Build a Docker Image') {
            steps {
                script {
                    dockerapp = docker.build("viniciusemanuelds/guia-jenkins:${env.BUILD_ID}",'-f ./src/Dockerfile ./src')
                }
            }
        }

        stage('Push a Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy no Kubernetes') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }

            agent {
                docker {
                    image: 'jenkins/inbound-agent:latest-alpine'
                    args: '-u root'
                }
            }

            steps {
                sh '''
                    echo "Instalando dependências..."
                    apk add --no-cache curl
                    
                    echo "Baixando e instalando kubectl..."
                    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
                    
                    echo "Verificando a instalação:"
                    kubectl version --client
                '''
                
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
                    sh 'kubectl apply -f ./k8s/deployment.yaml'
                }
            }
        }
    }
}
