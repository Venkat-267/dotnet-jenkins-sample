pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1' // AWS Region where Elastic Beanstalk is deployed
        S3_BUCKET = 'nodejs-artifact-venkat ' // S3 Bucket name to upload application package
        EB_APP_NAME = 'DotNetJenkinsSample' // Elastic Beanstalk application name
        EB_ENV_NAME = 'DotNetJenkinsSample-env' // Elastic Beanstalk environment name
    }

    stages {
        stage('Checkout') {
            steps {
                // Check out the code from your SCM
                git branch: 'master', url: 'https://github.com/your-username/your-dotnet-app.git'
            }
        }

        stage('Build') {
            steps {
                // Build the .NET application
                sh 'dotnet build --configuration Release'
            }
        }

        stage('Test') {
            steps {
                // Run unit tests
                sh 'dotnet test --no-build --configuration Release'
            }
        }

        stage('Publish') {
            steps {
                // Publish the application to a folder
                sh 'dotnet publish -c Release -o ./publish'
            }
        }

        stage('Package') {
            steps {
                // Zip the published output
                sh 'zip -r app.zip ./publish/*'
            }
        }

        stage('Upload to S3') {
            steps {
                // Upload the package to S3
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials-id',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                    aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                    aws configure set region $AWS_REGION
                    aws s3 cp app.zip s3://$S3_BUCKET/app.zip
                    '''
                }
            }
        }

        stage('Deploy to Elastic Beanstalk') {
            steps {
                // Deploy the application to Elastic Beanstalk
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials-id',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh '''
                    aws elasticbeanstalk create-application-version --application-name $EB_APP_NAME --version-label v$BUILD_NUMBER --source-bundle S3Bucket=$S3_BUCKET,S3Key=app.zip
                    aws elasticbeanstalk update-environment --environment-name $EB_ENV_NAME --version-label v$BUILD_NUMBER
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
