@Library('jenkins-shared-library')_

pipeline {
    agent any

    environment {
        REGION = 'ap-south-1'
        REGISTRY = '123456789012.dkr.ecr.ap-south-1.amazonaws.com'
        REPO = 'reporting-backend'
        TAG = "${BRANCH_NAME}-${BUILD_NUMBER}-${GIT_COMMIT.take(7)}"
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
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    dockerBuildPush(
                        image: "${REGISTRY}/${REPO}",
                        tag: TAG,
                        region: REGION,
                        registry: REGISTRY
                    )
                }
            }
        }
    }
}
