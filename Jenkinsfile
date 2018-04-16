pipeline {
  environment {
    FUGUE_USER_NAME = credentials("FUGUE_USER_NAME")
    FUGUE_USER_SECRET = credentials("FUGUE_USER_SECRET")
    FUGUE_RBAC_DO_AS = "true"
    FUGUE_LWC_OPTIONS = "true"
    AWS_DEFAULT_REGION = "us-east-1"
    EWC_DNSNAME = "ewc-api.fugue.cloud"
    EWC_USER_NAME = "jonathan@fugue.co"
    EWC_USER_PASS = "asdfasdf"
  }
  agent {
    docker {
      image "225195660222.dkr.ecr.us-east-1.amazonaws.com/fugue/client:latest"
      registryUrl "https://225195660222.dkr.ecr.us-east-1.amazonaws.com/fugue/client"
      registryCredentialsId "ecr:us-east-1:ECS_REPO"
    }
  }
  stages {
    stage("Validate Policy") {
      steps {
        sh "lwc Policy/AWSCISFoundationsBenchmark.lw"
      }
    }
    stage("Approve Policy") {
      when {
        branch "master"
      }
      steps {
        input "Please review and approve this change"
      }
    }
    stage("Apply Policy") {
      when {
        branch "master"
      }
      steps {
        sh "lwc -s snapshot Policy/AWSCISFoundationsBenchmark.lw -o AWSCISFoundationsBenchmark.tar.gz"
        sh '''
          set +x
          TOKEN=$(curl -s -X POST "https://$EWC_DNSNAME/oidc/token" \
            -H "content-type: application/x-www-form-urlencoded" \
            --data "username=$EWC_USER_NAME&password=$EWC_USER_PASS&client_id=fugue_enterprise_web_console&grant_type=password" \
            | jq -r .access_token )
          curl -s https://$EWC_DNSNAME/validations?name=AWSCISFoundationsBenchmark \
            -F "snapshot=@AWSCISFoundationsBenchmark.tar.gz" \
            -H "accept: application/json" \
            -H "authorization: Bearer $TOKEN"
        '''
      }
    }
  }
}
