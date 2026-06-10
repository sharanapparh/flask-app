pipeline {

    agent any

    environment {

        DOCKER_IMAGE = "your-dockerhub-username/flask-app"
        DOCKER_TAG = "${BUILD_NUMBER}"

    }

    tools {
        jdk 'jdk17'
    }

    stages {

        stage('Checkout Code') {

            steps {

                echo "Cloning Source Code..."

                git branch: 'main',
                url: 'https://github.com/sharanapparh/flask-app.git        }
        }

        stage('SonarQube Analysis') {

            steps {

                echo "Running SonarQube Scan..."

                withSonarQubeEnv('Sonar') {

                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=flask-app \
                    -Dsonar.projectName=flask-app \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://SONAR_SERVER_IP:9000 \
                    -Dsonar.token=YOUR_SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {

            steps {

                timeout(time: 5, unit: 'MINUTES') {

                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {

            steps {

                echo "Building Docker Image..."

                sh '''
                docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                '''
            }
        }

        stage('Docker Login') {

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
                    '''
                }
            }
        }

        stage('Push Docker Image') {

            steps {

                echo "Pushing Docker Image..."

                sh '''
                docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                '''
            }
        }

        stage('Deploy to Kubernetes') {

            steps {

                echo "Deploying Application..."

                sh '''
                kubectl set image deployment/flask-app \
                flask-app=${DOCKER_IMAGE}:${DOCKER_TAG}

                kubectl rollout status deployment/flask-app
                '''
            }
        }

    }

    post {

        success {

            echo "Pipeline Completed Successfully"
        }

        failure {

            echo "Pipeline Failed"
        }

        always {

            sh 'docker image prune -f'
        }
    }
}
