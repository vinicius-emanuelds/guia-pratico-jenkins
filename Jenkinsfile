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
            steps {
                sh 'echo "Executando o comando kubectl apply"'
            }
        }
    }
}