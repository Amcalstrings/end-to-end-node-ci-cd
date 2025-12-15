pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        REPOSITORY_URI = "717279705656.dkr.ecr.us-east-1.amazonaws.com/node-repo"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    tools {
        nodejs 'node18'
        jdk 'jdk17'
    }

    stages{
        stage ('Checkout Code'){
            steps {
                checkout scm
            }
        }

        stage('SonarCloud Scan') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                    docker run --rm \
                    -e SONAR_TOKEN=$SONAR_TOKEN \
                    -v $(pwd):/usr/src \
                    sonarsource/sonar-scanner-cli \
                    -Dsonar.projectKey=amcal-org_node_project \
                    -Dsonar.organization=amcal-org \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=https://sonarcloud.io
                    '''
                }
            }
        }

        stage ('Login to ECR'){
            steps {
                withCredentials([aws(credentialsId: 'AWS-ECR-CRED', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_DEFAULT_REGION \
                        | docker login --username AWS --password-stdin $REPOSITORY_URI
                    '''
                }
            }
        }

        stage ('Build Docker Image') {
            steps{
                sh '''
                    docker build -t my-app:$IMAGE_TAG app/
                    docker my-app:$IMAGE_TAG $REPOSITORY_URI:$IMAGE_TAG
                '''
            }
        }

        stage ('PUSH to ECR'){
            steps {
                sh '''
                    docker push $REPOSITORY_URI:$IMAGE_TAG
                '''
            }
        }
    }
    post {
        success {
            echo '✅ Sonarcloud analysis successful'
            echo '✅ Build and Docker Image Push successful!'
            echo "Pushed Image: $REPOSITORY_URI:$IMAGE_TAG"
        }
        failure {
            echo '❌ Build failed. Check logs above'
        }
    }
}