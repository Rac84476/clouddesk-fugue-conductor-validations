pipeline {
  environment {
    FUGUE_USER_NAME = credentials("FUGUE_USER_NAME")
    FUGUE_USER_SECRET = credentials("FUGUE_USER_SECRET")
    FUGUE_RBAC_DO_AS = "true"
    FUGUE_LWC_OPTIONS = "true"
    AWS_DEFAULT_REGION = "us-east-1"
    EWC_DNSNAME = "ewc-api.fugue.cloud"
    EWC_USER_NAME = credentials("EWC_USER_NAME")
    EWC_USER_PASS = credentials("EWC_USER_PASS")
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
        sh "lwc -s snapshot Policy/AWSCISFoundationsBenchmark.lw -o AWSCISFoundationsBenchmark.tar.gz"
      }
    }
    stage("Apply Policy") {
      when {
        branch "master"
      }
      steps {
        script {
          def token = sh(script: 'curl -s -X POST "https://$EWC_DNSNAME/oidc/token" \
            -H "content-type: application/x-www-form-urlencoded" \
            --data "username=$EWC_USER_NAME&password=$EWC_USER_PASS&client_id=fugue_enterprise_web_console&grant_type=password" \
            | jq -r .access_token', returnOutput: true)

          echo $token
        }
        /*
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
        */
      }
    }
  }
}
