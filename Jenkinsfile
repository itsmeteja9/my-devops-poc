pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "itsmeteja9/devops-poc-app"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Test') {
            steps {
                dir('app') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        }
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.image(DOCKER_IMAGE).push('latest')
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
