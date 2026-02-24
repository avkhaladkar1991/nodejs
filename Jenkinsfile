pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "avkhaladkar1991/nodejs-devops-app:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
          steps {
           git credentialsId: 'github-creds',
            url: 'https://github.com/avkhaladkar1991/nodejs.git',
            branch: 'main'
      }
    }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Push') {
         steps {
           withCredentials([usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push ${IMAGE_NAME}:${IMAGE_TAG}
          '''
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
                withCredentials([usernamePassword(
                 credentialsId: 'github-creds',
                 usernameVariable: 'GIT_USER',
                 passwordVariable: 'GIT_TOKEN'
        )])
                sh """
                 git config user.email "jenkins@ci.com"
                 git config user.name "jenkins"
                 git add .
                 git commit -am "CI: update image tag ${IMAGE_TAG}" || true
                 git push
                """
                git credentialsId: 'github-creds',
                url: 'https://github.com/avkhaladkar1991/nodejs.git',
                branch: 'main'
            }
        }
    }
}
