pipeline {
    agent any

    environment {
        IMAGE_NAME = "manaslenovo/my-cicd-project"
        CONTAINER_NAME = "my-website"
    }

    stages {

        stage('Pull Code from GitHub') {
            steps {
                checkout scm
                echo 'Code pulled successfully!'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
                sh 'docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest'
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$BUILD_NUMBER
                        docker push $IMAGE_NAME:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy Container Locally') {
            steps {
                sh '''
                    docker rm -f $CONTAINER_NAME || true

                    docker run -d \
                    --name $CONTAINER_NAME \
                    --restart unless-stopped \
                    -p 8080:80 \
                    $IMAGE_NAME:latest
                '''
            }
        }

    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
            echo 'Application: http://localhost:8080'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins Console Output.'
        }
    }
}
