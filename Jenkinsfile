pipeline {
    agent any

    environment {
        IMAGE_NAME = "avkhaladkar1991/nodejs-devops-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/avkhaladkar1991/nodejs.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Update Helm Values') {
            steps {
                sh """
                    sed -i '' 's/tag:.*/tag: "${IMAGE_TAG}"/' helm/nodejs-app/values.yaml
                """
            }
        }

        stage('Commit & Push Helm Update') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh """
                        git config user.email "jenkins@ci.com"
                        git config user.name "jenkins"
                        git add helm/nodejs-app/values.yaml
                        git commit -m "CI: update image tag ${IMAGE_TAG}" || true
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/avkhaladkar1991/nodejs.git HEAD:main
                    """
                }
            }
        }
    }
}
