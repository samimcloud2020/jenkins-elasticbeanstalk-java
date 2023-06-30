pipeline {
    agent any 
     environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
        ARTIFACT_NAME = 'calculator.jar'
        AWS_S3_BUCKET = 'somesh123456'
        AWS_EB_APP_NAME = 'app1'
        AWS_EB_ENVIRONMENT = 'App1-env'
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "10.128.0.32:8081"
        NEXUS_REPOSITORY = "vprofile-release"
	    NEXUS_REPO_ID = "vprofile-release"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
        ARTVERSION = "${env.BUILD_ID}"
		PROJECT_ID = 'genuine-fold-316617'
        CLUSTER_NAME = 'stagging'
        CLUSTER_NAME1 = "production"
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'multi-k8s' 
         
    }
    stages {
      stage('Build') {
            // build stage
        }
        stage('Test') {
           // test stage
        }
      stage('Publish') {
            steps {
                sh './mvnw package'
                // bat '.\mvnw package'
            }
            post {
                success {
                    archiveArtifacts 'target/*.jar'
                    sh 'aws configure set region us-east-1'
                    sh 'aws s3 cp ./target/calculator-0.0.1-SNAPSHOT.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME'
                    sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
                    sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
                }
            }
        }
