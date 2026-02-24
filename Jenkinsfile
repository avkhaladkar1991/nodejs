pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yourdockerhub/nodejs-devops-app:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/yourrepo/nodejs-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([ credentialsId: 'dockerhub-creds', url: '' ]) {
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Update Helm Values') {
            steps {
                sh """
                sed -i 's/tag:.*/tag: "${BUILD_NUMBER}"/' helm/nodejs-app/values.yaml
                """
            }
        }

        stage('Commit & Push Helm Update') {
            steps {
                sh """
                git config user.email "devops@example.com"
                git config user.name "devops"
                git add .
                git commit -m "Update image tag to ${BUILD_NUMBER}"
                git push
                """
            }
        }
    }
}
