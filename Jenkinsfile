@Library('jenkins-shared-library')_

pipeline {
    agent any

    environment {
        REGION   = 'ap-south-1'
        REGISTRY = '722396409432.dkr.ecr.ap-south-1.amazonaws.com'
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

        /* ---------- NEW STAGE 1 ---------- */
        stage('AWS ECR Login') {
            when {
                branch 'dev'
            }
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''#!/bin/sh
set -e
aws --version
aws ecr get-login-password --region "$REGION" \
| docker login --username AWS --password-stdin "$REGISTRY"
'''
                }
            }
        }

        /* ---------- NEW STAGE 2 ---------- */
        stage('Docker Build Image') {
            when {
                branch 'dev'
            }
            steps {
                sh '''#!/bin/sh
set -e
docker build -t "$IMAGE" .
'''
            }
        }

        /* ---------- EXISTING PUSH STAGE ---------- */
        stage('Docker Push Image') {
            when {
                branch 'dev'
            }
            steps {
                sh '''#!/bin/sh
set -e
docker push "$IMAGE"
'''
            }
        }
    }
}
