node {

  /* Setup our environment */
  withEnv(["AWS_DEFAULT_REGION=us-east-1",
           "FUGUE_RBAC_DO_AS=1",
           "FUGUE_LWC_OPTIONS=true"]) {

    /* Checkout git repo */
    checkout(scm)

    /* Use Amazon ECR repo */
    docker.withRegistry("https://225195660222.dkr.ecr.us-east-1.amazonaws.com/fugue/client", "ecr:us-east-1:ECS_REPO" ) {

      /* Pull the fugue client docker container from ECR */
      docker.image("225195660222.dkr.ecr.us-east-1.amazonaws.com/fugue/client:latest").inside {

        /* Set our Fugue credentials */
        withCredentials([string(credentialsId: "FUGUE_USER_NAME", variable: "FUGUE_USER_NAME"),
                         string(credentialsId: "FUGUE_USER_SECRET", variable: "FUGUE_USER_SECRET")]) {

          /* Validate that the policy compiles */
          stage("Validate Policy") {
            sh(script: "lwc Policy/AWSCISFoundationsBenchmark.lw")
          }

          /* Snapshot the policy */
          stage("Snapshot Policy") {
            sh(script: "lwc -s snapshot Policy/AWSCISFoundationsBenchmark.lw -o AWSCISFoundationsBenchmark.tar.gz")
          }

          /* Publish the policy */
          stage("Publish Policy") {
            sh '''
              ls -l
              whoami
            '''
            sh '''
              /* set +x */
              TOKEN=$(curl -s -X POST "https://$EWC_DNSNAME/oidc/token" \
                -H "content-type: application/x-www-form-urlencoded" \
                --data "username=$EWC_USER_NAME&password=$EWC_USER_PASS&client_id=fugue_enterprise_web_console&grant_type=password" \
                | jq -r .access_token)
              curl -s -X POST https://$EWC_DNSNAME/validations?name=AWSCISFoundationsBenchmark \
                -H "accept: application/json" \
                -H "authorization: Bearer $TOKEN" \
                -F "snapshot=@AWSCISFoundationsBenchmark.tar.gz"
            '''
          }
        }
      }
    }
  }
}
