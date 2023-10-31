pipeline {
  agent {
    docker {
      image 'public.ecr.aws/sam/build-python3.9:latest'
      args '-u root:root'
    }
  }
  environment {
    DEV_SAM_CONFIGFILE = 'samconfig.toml'
    DEV_SAM_TEMPLATE = 'template.yaml'
    DEV_STACK_NAME = 'cross-account-sam'
    // DEV_PIPELINE_EXECUTION_ROLE = Use Jenkins EC2 IAM Role
    DEV_CLOUDFORMATION_EXECUTION_ROLE = 'Dev Account IAM Role ARN' // Dev Account IAM Role ARN
    MGMT_S3_BUCKET = 'MGMT S3 Bucket Name'
    MGMT_S3_BUCKET_PREFIX = 'MGMT S3 Bucket Prefix'
    DEV_REGION = 'ap-northeast-2'
  }
  stages {

    stage('Environment Check') {
      steps {
        sh '''
          env
          
          echo \"# AWS Version\"
          aws --version

          echo \"# AWS STS Token Check\"
          aws sts get-caller-identity

          echo \"# SAM Version\"
          sam --version

          echo \"# Python Version\"
          python3 --version
          pip3 --version

          echo \"# Working Directory\"
          pwd
        '''
      }
    }

    stage('SAM Validation') {
      steps {
        sh '''
        echo \"# SAM Template Validation Check.\"

        sam validate --lint --template ${DEV_SAM_TEMPLATE}
        '''
      }
    }

    stage('SAM Build - DEV') {
      steps {
        sh '''
        echo \"# SAM Template Build to CloudFormation Format.\"

        sam build --template ${DEV_SAM_TEMPLATE}
        '''
        }
      }

    stage('SAM Deploy - DEV') {
      steps {
        sh '''
        echo \"# Get AWS STS Credential from Cross Account.\"
        CREDENTIAL=\$(aws sts assume-role \
          --duration-seconds 900 \
          --role-arn ${DEV_CLOUDFORMATION_EXECUTION_ROLE} \
          --role-session-name SAMSession \
          --output text \
          --query \'Credentials.[AccessKeyId,SecretAccessKey,SessionToken,Expiration]\')
          
        export AWS_ACCESS_KEY_ID=$(echo $CREDENTIAL | awk '{print $1}')
        export AWS_SECRET_ACCESS_KEY=$(echo $CREDENTIAL | awk '{print $2}')
        export AWS_SESSION_TOKEN=$(echo $CREDENTIAL | awk '{print $3}')
        export SESSION_EXPIRATION=$(echo $CREDENTIAL | awk '{print $4}')

        aws sts get-caller-identity

        echo \"# SAM Template Deploy to CloudFormation Stacks.\"

        sam deploy \
          --config-file ${DEV_SAM_CONFIGFILE} \
          --template ${DEV_SAM_TEMPLATE} \
          --region ${DEV_REGION} \
          --s3-bucket ${MGMT_S3_BUCKET} \
          --s3-prefix ${MGMT_S3_BUCKET_PREFIX}
        '''
      }
    }
  }
}