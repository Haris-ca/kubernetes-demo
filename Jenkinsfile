
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOCKERHUB_NAMESPACE = 'harisca03'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm

                sh '''
                    echo "Checking project structure..."
                    test -f admin/Dockerfile
                    test -f frontend/Dockerfile
                    test -f backend/Dockerfile
                    echo "All Dockerfiles found!"
                '''
            }
        }

        stage('Build Admin Image') {
            steps {
                sh '''
                    docker build \
                    -t ${DOCKERHUB_NAMESPACE}/admin:${BUILD_NUMBER} \
                    -t ${DOCKERHUB_NAMESPACE}/admin:latest \
                    ./admin
                '''
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh '''
                    docker build \
                    -t ${DOCKERHUB_NAMESPACE}/frontend:${BUILD_NUMBER} \
                    -t ${DOCKERHUB_NAMESPACE}/frontend:latest \
                    ./frontend
                '''
            }
        }

        stage('Build Backend Image') {
            steps {
                sh '''
                    docker build \
                    -t ${DOCKERHUB_NAMESPACE}/backend:${BUILD_NUMBER} \
                    -t ${DOCKERHUB_NAMESPACE}/backend:latest \
                    ./backend
                '''
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x

                        echo "$DOCKER_TOKEN" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push ${DOCKERHUB_NAMESPACE}/admin:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_NAMESPACE}/admin:latest

                        docker push ${DOCKERHUB_NAMESPACE}/frontend:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_NAMESPACE}/frontend:latest

                        docker push ${DOCKERHUB_NAMESPACE}/backend:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_NAMESPACE}/backend:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'All three microservice images pushed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }

        always {
            sh 'docker logout || true'
        }
    }
}
