
pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        BUCKET     = 'dine-test-vc-jan'
        ENV        = 'dev'
        REPO_URL   = 'https://github.com/rrj-bit-211/dine-cicd-test.git' // <-- change
        BRANCH     = 'master'                                      // <-- or main
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: env.BRANCH, url: env.REPO_URL
            }
        }

        stage('Determine Version (Commit SHA)') {
            steps {
                script {
                    env.VERSION = sh(
            script: 'git rev-parse --short HEAD',
            returnStdout: true
          ).trim()
                    echo "Version (Commit SHA): ${env.VERSION}"
                }
            }
        }
   
        stage('Upload to S3 (Versioned Path)') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-poc-creds']]) {
                    sh """
        set -e
        export AWS_DEFAULT_REGION=${AWS_REGION}

        if [ -d sql ]; then
          aws s3 sync sql/ s3://${BUCKET}/${ENV}/${VERSION}/sql/ --delete
        else
          echo "Skipping: sql/ not found"
        fi

        if [ -d parameters ]; then
          aws s3 sync parameters/ s3://${BUCKET}/${ENV}/${VERSION}/parameters/ --delete
        else
          echo "Skipping: parameters/ not found"
        fi
      """
                }
            }
        }


        stage('Verify Upload') {
            steps {
                sh """
          export AWS_DEFAULT_REGION=${AWS_REGION}
          aws s3 ls s3://${BUCKET}/${ENV}/${VERSION}/sql/
          aws s3 ls s3://${BUCKET}/${ENV}/${VERSION}/parameters/
        """
            }
        }
    }

    post {
        success {
            echo "POC upload completed for version ${env.VERSION}"
        }
        failure {
            echo 'POC upload failed. Check console logs for details.'
        }
    }
}
