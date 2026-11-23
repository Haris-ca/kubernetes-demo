
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOCKERHUB_NAMESPACE = 'harisca03'
        GITOPS_REPO = 'https://github.com/Haris-ca/kubernetes-demo-gitops.git'
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

        stage('Update GitOps Repository') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-gitops-credentials',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )
                ]) {
                    dir('gitops') {
                        sh '''
                            set -eu
                            set +x

                            echo "Cloning GitOps repository..."

                            git clone --branch main "$GITOPS_REPO" .

                            echo "Updating Kubernetes image tags..."

                            sed -i -E \
                              "s|(image: ${DOCKERHUB_NAMESPACE}/admin:).*|\\1${BUILD_NUMBER}|" \
                              shoplite/admin-deployment.yaml

                            sed -i -E \
                              "s|(image: ${DOCKERHUB_NAMESPACE}/frontend:).*|\\1${BUILD_NUMBER}|" \
                              shoplite/frontend-deployment.yaml

                            sed -i -E \
                              "s|(image: ${DOCKERHUB_NAMESPACE}/backend:).*|\\1${BUILD_NUMBER}|" \
                              shoplite/backend-deployment.yaml

                            echo "Updated image references:"
                            grep -n 'image:' shoplite/*deployment.yaml

                            git config user.name "Jenkins CI"
                            git config user.email "jenkins@example.com"

                            git add shoplite/*deployment.yaml

                            git commit -m "Update Shoplite images to build ${BUILD_NUMBER}"

                            cat > .git-askpass.sh <<'EOF'
#!/bin/sh
case "$1" in
    *Username*) echo "$GIT_USER" ;;
    *Password*) echo "$GIT_TOKEN" ;;
esac
EOF

                            chmod 700 .git-askpass.sh
                            trap 'rm -f .git-askpass.sh' EXIT

                            export GIT_ASKPASS="$PWD/.git-askpass.sh"
                            export GIT_TERMINAL_PROMPT=0

                            echo "Pushing changes to GitOps repository..."
                            git push origin main

                            echo "GitOps repository updated successfully!"
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'CI successful! Images pushed and GitOps manifests updated.'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }

        always {
            sh 'docker logout || true'
        }
    }
}
