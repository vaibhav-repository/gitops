pipeline {

    agent any

    environment {
        AWS_REGION = 'ap-south-1'

        ECR_REPO = '255834079384.dkr.ecr.ap-south-1.amazonaws.com/production/gitops-repo'

        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE = "${ECR_REPO}:${IMAGE_TAG}"

        GITOPS_REPO = 'https://github.com/vaibhav-repository/gitops-main.git'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    npm install
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    npm test
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t ${IMAGE} .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-new-cred']
                ]) {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login \
                        --username AWS \
                        --password-stdin ${ECR_REPO}
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push ${IMAGE}
                '''
            }
        }

        stage('Update Helm Values') {
            steps {
                sh '''
                    rm -rf gitops

                    git clone ${GITOPS_REPO} gitops

                    cd gitops/nodejs-app

                    sed -i "s/^  tag:.*/  tag: \\"${IMAGE_TAG}\\"/" values.yaml

                    git config user.name "Jenkins"
                    git config user.email "jenkins@example.com"

                    git add values.yaml

                    git commit \
                    -m "Update nodejs-app image to ${IMAGE_TAG}" || true

                    git push origin main
                '''
            }
        }
    }
}