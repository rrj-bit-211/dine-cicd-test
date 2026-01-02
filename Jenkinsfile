pipeline {
  agent any

  environment {
    AWS_REGION = "ap-south-1"
    BUCKET     = "dine-test-vc-jan"  // If BUCKET comes from your ts3/env, keep this var name and value injected at runtime
  }

  stages {

    stage('Checkout Code') {
      steps {
        // Change branch to 'main' if your repo uses main
        git branch: 'master',
            url: 'https://github.com/rrj-bit-211/dine-cicd-test.git'
      }
    }

    stage('Preflight Checks') {
      steps {
        sh """
          set -e
          command -v aws >/dev/null 2>&1 || { echo 'AWS CLI not found on agent PATH'; exit 1; }
          command -v git >/dev/null 2>&1 || { echo 'Git not found on agent PATH'; exit 1; }

          # Confirm source folder exists in the checked-out repo
          test -d de-etl-prod/pos/guestcheck || { echo 'Missing folder: de-etl-prod/pos/guestcheck'; exit 1; }
        """
      }
    }

    stage('First-time Upload to S3 (No Versioning)') {
      steps {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: 'aws-poc-creds',
          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
          sh """
            set -e
            aws sts get-caller-identity
            # Sync the specific folder to the bucket root path (no ENV/version prefix)
            aws s3 sync de-etl-prod/pos/guestcheck/ \
              s3://${BUCKET}/de-etl-prod/pos/guestcheck/ \
              --region ${AWS_REGION} \
              --exclude ".git/*" --exclude ".github/*" --exclude ".gitignore"

            echo "Uploaded: de-etl-prod/pos/guestcheck -> s3://${BUCKET}/de-etl-prod/pos/guestcheck/"
          """
        }
      }
    }

    stage('Verify Upload') {
      steps {
       withCredentials([[
        $class: 'AmazonWebServicesCredentialsBinding',
        credentialsId: 'aws-poc-creds',
        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
    ]]) {  
        sh """
          aws s3 ls s3://${BUCKET}/de-etl-prod/pos/guestcheck/ --recursive --region ${AWS_REGION}
        """
      }
    }
  }
  }

  post {
    success { echo "✅ First-time upload finished: s3://${BUCKET}/de-etl-prod/pos/guestcheck/" }
    failure { echo "❌ Upload failed. Check agent tools, IAM permissions, repo path, or bucket name." }
  }
}
