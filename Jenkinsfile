pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        REPOSITORY_URI = "717279705656.dkr.ecr.us-east-1.amazonaws.com/node-repo"
    }

    tools {
        nodejs 'node18'
        jdk 'jdk17'
        sonarScanner 'sonar-scanner'
    }

    stages{
        stage ('Checkout Code'){
            steps {
                checkout scm
            }
        }

        stage ('RunSonarCloudAnalysis'){
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=amcal-org_node_project \
                        -Dsonar.organization=amcal-org \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Sonarcloud analysis successful'
        }
        failure {
            echo '❌ Build failed. Check logs above'
        }
    }
}