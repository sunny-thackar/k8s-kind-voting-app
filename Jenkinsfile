pipeline {
    agent any

    environment {
        VOTE_IMAGE = "sunnythacker/examplevotingapp_vote"
        RESULT_IMAGE = "sunnythacker/examplevotingapp_result"
        TAG = "${BUILD_NUMBER}"
    }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/YOUR_USERNAME/YOUR_REPO.git'
            }
        }

        stage('Docker Login') {
            steps {
                sh """
                echo $DOCKER_PASSWORD | docker login \
                -u $DOCKER_USERNAME \
                --password-stdin
                """
            }
        }

        stage('Build Images') {
            steps {

                dir('vote') {
                    sh """
                    docker build -t $VOTE_IMAGE:$TAG .
                    docker tag $VOTE_IMAGE:$TAG $VOTE_IMAGE:latest
                    """
                }

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
