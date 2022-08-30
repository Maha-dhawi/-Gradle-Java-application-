pipeline {
    agent any
    environment { //variables

        AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')

        AWS_S3_BUCKET = "gradle-maha"
        ARTIFACT_NAME = "gradle-0.0.1-SNAPSHOT.jar"
        AWS_EB_APP_NAME = "gradle-maha"
        AWS_EB_APP_VERSION = "${BUILD_ID}" // when you want to roll back
        AWS_EB_ENVIRONMENT = "Gradlemaha-env"

    }
    stages {
        stage('Validate') {
            steps {
                sh "chmod +x gradlew"
                sh "./gradlew clean"


            }
        }
// 1.Compile the code (This should create an artifact)
         stage('Build') {
            steps {
                
                sh "./gradlew build"

            }
            post {
                always {
                    junit '**/build/test-results/test/TEST-*.xml' //it is from the syntax genertor snd ** -> means anything in the Start
                }
                success { //when the step is successful archive the artfact
                    archiveArtifacts artifacts: '**/build/libs/**.jar', followSymlinks: false    
                }
            }
        }


      
        //5. Publish the artifacts
        stage('Publish artefacts to S3 Bucket') {
            steps {

                sh "aws configure set region us-east-1"

                sh "aws s3 cp ./build/libs/**.jar s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"
                
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
