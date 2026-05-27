pipeline {
    agent any

    environment {
        VOTE_IMAGE = "sunnythacker/examplevotingapp_vote"
        RESULT_IMAGE = "sunnythacker/examplevotingapp_result"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/sunny-thackar/k8s-kind-voting-app.git'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh """
                    echo \$DOCKER_PASS | docker login \
                    -u \$DOCKER_USER \
                    --password-stdin
                    """
                }
            }
        }

        stage('Build Vote Image') {
            steps {
                dir('vote') {
                    sh """
                    docker build -t $VOTE_IMAGE:$TAG .
                    docker tag $VOTE_IMAGE:$TAG $VOTE_IMAGE:latest
                    """
                }
            }
        }

        stage('Build Result Image') {
            steps {
                dir('result') {
                    sh """
                    docker build -t $RESULT_IMAGE:$TAG .
                    docker tag $RESULT_IMAGE:$TAG $RESULT_IMAGE:latest
                    """
                }
            }
        }

        stage('Push Images') {
            steps {
                sh """
                docker push $VOTE_IMAGE:$TAG
                docker push $VOTE_IMAGE:latest

                docker push $RESULT_IMAGE:$TAG
                docker push $RESULT_IMAGE:latest
                """
            }
        }

        stage('Deploy to KIND') {
            steps {
                sh """
                kubectl apply -f k8s/vote-deployment.yml
                kubectl apply -f k8s/result-deployment.yml

                kubectl rollout restart deployment vote
                kubectl rollout restart deployment result
                """
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }

        failure {
            echo 'Pipeline Failed!'
        }
    }
}
