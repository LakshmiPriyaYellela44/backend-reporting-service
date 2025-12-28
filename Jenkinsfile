@Library('jenkins-shared-library')_

pipeline {
    agent any

    environment {
        REGION   = 'ap-south-1'
        REGISTRY = '123456789012.dkr.ecr.ap-south-1.amazonaws.com'
        REPO     = 'reporting-backend'
        TAG      = "${BRANCH_NAME}-${BUILD_NUMBER}-${GIT_COMMIT.take(7)}"
        IMAGE    = "${REGISTRY}/${REPO}:${TAG}"
    }

    tools {
        jdk 'JDK_17'
        maven 'Maven_3'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Docker Build & Push') {
            when {
                branch 'dev'
            }

            agent {
                docker {
                    image 'docker:27.0.3-cli'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }

            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                      apk add --no-cache aws-cli

                      aws ecr get-login-password --region $REGION \
                      | docker login --username AWS --password-stdin $REGISTRY

                      docker build -t $IMAGE .
                      docker push $IMAGE
                    '''
                }
            }
        }
    }
}
