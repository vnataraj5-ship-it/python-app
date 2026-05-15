pipeline {

    agent any

    environment {

        IMAGE_NAME = "python-app"

        NEXUS_REGISTRY = "13.126.233.214:8082"

        SONAR_SCANNER = tool 'sonar-scanner'
    }

    stages {

        stage('Checkout') {

            steps {

                git 'https://github.com/vnataraj5-ship-it/python-app.git'
            }
        }

        stage('SonarQube Analysis') {

            steps {

                withSonarQubeEnv('sonar-server') {

                    sh """
                    ${SONAR_SCANNER}/bin/sonar-scanner \
                    -Dsonar.projectKey=python-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://13.126.233.214:9000
                    """
                }
            }
        }

        stage('Docker Build') {

            steps {

                sh """
                docker build -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Push Docker Image to Nexus') {

            steps {

                withCredentials([usernamePassword(
                    credentialsId: 'nexus-cred',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {

                    sh """
                    docker login ${NEXUS_REGISTRY} -u $USERNAME -p $PASSWORD

                    docker tag ${IMAGE_NAME}:latest \
                    ${NEXUS_REGISTRY}/repository/docker-hosted/${IMAGE_NAME}:latest

                    docker push \
                    ${NEXUS_REGISTRY}/repository/docker-hosted/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {

            steps {

                sh """
                export KUBECONFIG=/var/jenkins_home/.kube/config

                kubectl apply -f k8s/app-deploy-service.yaml
                """
            }
        }
    }
}
