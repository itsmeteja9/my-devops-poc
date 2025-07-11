pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "itsmeteja9/devops-poc-app"
        SONAR_SCANNER_HOME = tool 'sonarscanner'  // Must match Jenkins tool name
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonartoken', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('MySonarQube') {
                        dir('app') {
                            bat "${SONAR_SCANNER_HOME}\\bin\\sonar-scanner.bat " +
                                "-Dsonar.projectKey=devops-poc " +
                                "-Dsonar.sources=. " +
                                "-Dsonar.host.url=http://localhost:9000 " +
                                "-Dsonar.login=%SONAR_TOKEN%"
                        }
                    }
                }
            }
        }

        stage('Build & Test') {
            steps {
                dir('app') {
                    bat 'npm install'
                    bat 'npm test'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('app') {
                    script {
                        docker.build(DOCKER_IMAGE)
                    }
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    bat 'echo %PASS% | docker login -u %USER% --password-stdin'
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
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    script {
                        bat 'kubectl config current-context'
                        bat 'kubectl get nodes'
                        bat 'kubectl apply -f k8s\\deployment.yaml'
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
