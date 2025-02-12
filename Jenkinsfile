pipeline {
	agent any

	environment {

		AWS_ACCESS_KEY_ID     = credentials('aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')

		AWS_S3_BUCKET         = "golden-waste"
        AWS_REGION            = "me-south-1"

		ARTIFACT_NAME         = "ROOT.jar"

		SONAR_PROJECT_KEY     = "goldenwaste-sda-hackathon"
		SONAR_IP              = "3.84.249.189:9000"
		SONAR_TOKEN           = "sqp_0c014b47b2b7550a893e3e10a799ec0d1de4b235"
	}

	stages {

		
		stage('Validate') {
			steps {
				sh "mvn validate"
				sh "mvn clean"
			}
		}

		stage('Build') {
			steps {
				sh "mvn compile"
			}
		}

		stage('Test') {
			steps {
				sh "mvn test"
			}

			post {
				always {
					junit allowEmptyResults: true, testResults: '**/target/surefire-reports/TEST-*.xml'
				}
			}
		}

		stage('Quality Scan'){

			steps {
                sh '''
                mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                    -Dsonar.host.url=http://$SONAR_IP \
                    -Dsonar.login=$SONAR_TOKEN
                '''
            }
		}

		stage('Package') {
            steps {                
                sh "mvn package"
            }

            post {
                success {
                    archiveArtifacts artifacts: '**/target/**.jar', followSymlinks: false                   
                }
            }
        }

		stage('Publish artefacts to S3 Bucket') {
            steps {
                sh "aws configure set region $AWS_REGION"
                sh "aws s3 cp ./target/**.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"
            }
        }
	}
}