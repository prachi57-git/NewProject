pipeline {
    agent any

    environment {
        GIT_CREDENTIALS = credentials('github-token')
        AWS_CREDENTIALS = credentials('aws-credentials-id')
        S3_BUCKET = 'webpp-code-bucket'
        APP_NAME = 'WebAppDeploy'
        DEPLOYMENT_GROUP = 'WebApp-DeploymentGroup'
    }
    
    tools {
        maven 'maven 3.9.9'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/prachi57-git/NewProject.git', credentialsId: 'github-token'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload to S3') {
            steps {
                withAWS(credentials: AWS_CREDENTIALS) {
                    s3Upload(
                        bucket: S3_BUCKET,
                        file: 'target/NewProject.war'
                    )
                }
            }
        }

        stage('Deploy to EC2 via CodeDeploy') {
            steps {
                withAWS(credentials: AWS_CREDENTIALS) {
                    sh '''
                    aws deploy create-deployment \
                    --application-name ${APP_NAME} \
                    --deployment-group-name ${DEPLOYMENT_GROUP} \
                    --s3-location bucket=${S3_BUCKET},key=target/NewProject.war,bundleType=zip
                    '''
                }
            }
        }

        stage('Scale EC2') {
            steps {
                withAWS(credentials: AWS_CREDENTIALS) {
                    sh '''
                    aws autoscaling update-auto-scaling-group \
                    --auto-scaling-group-name my-auto-scaling-group \
                    --desired-capacity 3 \
                    --min-size 1 \
                    --max-size 5
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "CI/CD Pipeline completed!"
        }
    }
}

