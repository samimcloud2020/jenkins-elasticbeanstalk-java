pipeline {
    agent any 
     tools {
        maven "maven"
        jdk "java"
    	}	
     environment {
        AWS_EB_APP_NAME = 'app1'
        AWS_EB_ENVIRONMENT = 'env1'
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        ARTVERSION = "${env.BUILD_ID}"
        CREDENTIALS_ID = 'multi-k8s' 
	NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "10.5.0.4:8081"
        NEXUS_REPOSITORY = "vprofile-release"
	NEXUS_REPO_ID = "vprofile-release"
	NEXUS_CREDENTIAL_ID = "nexus3"
	AWS_REGION = "us-east-1"
	ECR_REGISTRY_ID = "291222035571.dkr.ecr.us-east-1.amazonaws.com/samim-repo1"
	EKS_CLUSTER_NAME1 = "cluster3333"
	EKS_CLUSTER_NAME2 = "cluster3334"
     
        BUCKET = "somesh123456"     
         
    }
 stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -s ./settings.xml '   //mvn clean install -DskipTests
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
		    }
            }
        }
	 stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

	    stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
	           
     	 stage('CODE ANALYSIS with SONARQUBE') {
          
		    environment {
                scannerHome = tool 'sonar'
          }

            steps {
                withSonarQubeEnv('sonar') {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=sonar \
                   -Dsonar.projectName=project1 \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }
                         
          }
        }
	stage(“Quality Gate”){

        timeout(time: 10, unit: ‘MINUTES’) {
              def qg= waitForQualityGate()
            if (qg.status!= ‘OK’){
                error “Pipeline aborted due to quality gate failure: ${qg.status}”
            }
        }         
              echo ‘Quality Gate Passed’

    }
stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTVERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } 
			        else {
				        error "*** File: ${artifactPath} , cloud not found";
			        }
                }
            }
        }
     stage ('Pull jar file from Nexus') {
        	steps {
		script{
                    withCredentials([usernameColonPassword(credentialsId: 'nexus3', variable: 'NEXUS_CREDENTIALS_ID')]) {
                    sh "rm -rf ${env.BUILD_ID}"
		    sh "mkdir ${env.BUILD_ID}"
		    sh "cd ${env.BUILD_ID}"
                    sh  'curl -u ${NEXUS_CREDENTIALS_ID} -o MavenTutorial.war http://10.5.0.4:8081/repository/vpro-maven-group/com/crunchify/MavenTutorial/${ARTVERSION}/MavenTutorial-${ARTVERSION}.war"'
                    sh "touch Dockerfile"
		    sh " echo FROM tomcat >> Dockerfile"
		    sh " echo ADD MavenTutorial.war /usr/local/tomcat/webapps >> Dockerfile"
	            sh " echo CMD catalina.sh run >> Dockerfile"
		    sh " echo EXPOSE 8080 >> Dockerfile"
				 }
			    }
			}
		  } 	     
	stage("Build image") {
            steps {
                script {
                    myapp = docker.build("samimbsnl/cicd:${env.BUILD_ID}")
                }
            }
        }
         stage("Push image to Dockerhub") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }        
   stage("Pull image to Dockerhub") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        sh " docker pull samimbsnl/cicd:${env.BUILD_ID}"
                        
                    }
                }
            }
        }
     stage('Push Docker Image to ECR') {
            steps {
                withAWS(credentials: 'aws', region: 'us-east-1') {
		script {	
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY_ID}"
		    sh "docker tag samimbsnl/cicd:${env.BUILD_ID}  291222035571.dkr.ecr.us-east-1.amazonaws.com/samim-repo1:${env.BUILD_ID}"
                    sh "docker push 291222035571.dkr.ecr.us-east-1.amazonaws.com/samim-repo1:${env.BUILD_ID}"
		 
                }
            }
        }
}
    stage('Push jar file to s3 bucket') {
            steps {
                withAWS(credentials: 'aws', region: 'us-east-1') {
			sh 'aws configure set region us-east-1'
                        sh 'aws s3 cp ./target/MavenTutorial-${ARTVERSION}.war s3://$BUCKET/MavenTutorial-${ARTVERSION}.war'
			}
		}
	}
	 stage ('mvn clean deploy'){
            steps {
                sh 'mvn clean deploy'
            }
            post {
                success {
                    echo 'sucessful deploy artifact to nexus'
                }
            }
        }  
/*
    stage('Integrate Jenkins with EKS Cluster and Deploy App in stagging cluster') {
            steps {
                withAWS(credentials: 'aws', region: 'us-east-1') {
                  script {
                    sh "aws eks update-kubeconfig --name ${EKS_CLUSTER_NAME1} --region ${AWS_REGION}"
                    sh "kubectl apply -f deploy.yaml"
                }
            }
        }
     }
   stage('Wait for SRE Approval') {
            steps{
                timeout(time:12, unit:'HOURS') {
                    input message:'Approve deployment?'
                }
            }
        }
   stage('Integrate Jenkins with EKS Cluster and Deploy App in production cluster') {
            steps {
                withAWS(credentials: 'aws', region: 'us-east-1') {
                  script {
                    sh "aws eks update-kubeconfig --name ${EKS_CLUSTER_NAME2} --region ${AWS_REGION}"
                    sh "kubectl apply -f deploy.yaml"
                }
            }
        }
     }	 
*/	 
      stage('Deploy to Elastic Beanstalk of war file') {
            steps {
                withAWS(credentials: 'aws', region: 'us-east-1') {
                  script {
                    sh 'aws configure set region us-east-1'
                    sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$BUCKET,S3Key=MavenTutorial-"${ARTVERSION}".war'
                    sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
                }
            }
        }
      }
 }
}

