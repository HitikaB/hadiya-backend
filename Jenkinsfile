pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '318988877498'
        AWS_DEFAULT_REGION = 'ap-south-1'
        ECR_REGISTRY = '318988877498.dkr.ecr.ap-south-1.amazonaws.com/hadiya-backend-hitika'
        WORKSPACE = "/var/lib/jenkins/workspace/hadiya-project/hadiya-backend"
        TAG = 'latest'
        ECS_CLUSTER = 'tf-cluster-hitika'
        SERVICE_NAME = 'hadiya-backend-svc'
        TASK_DEFINITION_NAME = 'hadiya-project'
        CONTAINER_NAME = 'hadiya'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    def repoUrl = 'https://github.com/HitikaB/hadiya-backend.git'
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [[$class: 'CloneOption', noTags: false, reference: '', shallow: true, timeout: 120]],
                        userRemoteConfigs: [[url: repoUrl]]
                    ])
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh 'sudo docker build -t hadiyabe:${BUILD_NUMBER} .'
                }
            }
        }

        stage('Pushing to ECR') {
            steps {
                script {
                    sh 'sudo su'
                    sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 318988877498.dkr.ecr.ap-south-1.amazonaws.com'
                    sh 'docker tag hadiyabe:${BUILD_NUMBER} 318988877498.dkr.ecr.ap-south-1.amazonaws.com/hadiya-backend-hitika:${BUILD_NUMBER}'
                    sh 'docker push 318988877498.dkr.ecr.ap-south-1.amazonaws.com/hadiya-backend-hitika:${BUILD_NUMBER}'
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    sh """
                        # Register the task definition
                        aws ecs register-task-definition --cli-input-json file://${WORKSPACE}/task-def.json
                        
                        # Update the service
                        aws ecs update-service \
                            --cluster $ECS_CLUSTER \
                            --service $SERVICE_NAME \
                            --task-definition $TASK_DEFINITION_NAME\
                    """
                }
            }
        }
    }
}
