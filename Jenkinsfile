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

          /* Apply policy to the Fugue Conductor */
          /* TO DO: Write validation-remove output to file and check for error */
          stage("Apply Policy") {
            def cmdStatusCode = sh(script: "fugue policy validation-remove AWSCISBenchmarks -y", returnStatus: true)
            if(cmdStatusCode == 0) {
              sh(script: "fugue policy validation-add Policy/AWSCISFoundationsBenchmark.lw --name AWSCISBenchmarks")
            } else {
              sh(script: "fugue policy validation-add Policy/AWSCISFoundationsBenchmark.lw --name AWSCISBenchmarks")
            }
          }
        }
      }
    }
  }
}
