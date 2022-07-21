pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
        
        AWS_S3_BUCKET = "artifact-bucket-repo-3"
        ARTIFACT_NAME = "pipelines-dotnet-core.dll"
        AWS_EB_APP_NAME = "dotnet-jenkins"
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "Dotnetjenkins-env"
        SONAR_IP = "54.226.50.200"
        SONAR_TOKEN = "sqp_6e8065683e55085b4e253e9af1329f96c9f3a3ca"
    }
    stages {
        stage('Restore') {
            steps {
                sh "dotnet restore"

            }
        }
        stage('Build') {
            steps {
                sh "dotnet build"
            }
        }
        stage('Test') {
            steps {
                sh "dotnet test"
            }
        }

        stage('Quality Scan'){
            steps {
                sh '''
                dotnet sonarscanner begin /k:"Online-cohort-project-2" /d:sonar.host.url="http://$SONAR_IP"  /d:sonar.login="$SONAR_TOKEN"
                dotnet build
                dotnet sonarscanner end /d:sonar.login="sqp_6e8065683e55085b4e253e9af1329f96c9f3a3ca"
                '''
            }
        }

        stage('Publish') {
            steps {
                sh "dotnet publish"
            }
            post {
                success {
                    archiveArtifacts artifacts: 'bin/Debug/net6.0/pipelines-dotnet-core.dll', followSymlinks: false
       
                }
            }
        }

        stage('Publish artefacts to S3 Bucket') {
            steps {

                sh "aws configure set region us-east-1"

                sh "aws s3 cp ./bin/Debug/net6.0/pipelines-dotnet-core.dll s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"
                
            }
        }

        stage('Deploy') {
            steps {

                sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'

                sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            
                
            }
        }
        

    }
}
