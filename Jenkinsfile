pipeline {
  agent any
  environment {
    PATH = "$PATH:/opt/apache-maven-3.9.8/bin"
  }
  stages {
    stage('Build and Reviewing Application') {
      steps {
        withSonarQubeEnv('sonarqube') {
          sh 'mvn clean package sonar:sonar'
        }
      }
    }
  }
  stage('SonarQube Analysis') {
  environment {
    // Tool name must match with Jenkins Tools for Sonar Scanner - Manage Jenkins >> Tools
    scannerHome = tool 'sonar-scanner'
  }
  steps {
    // Env value must match with the Sonar Server Name - Manage Jenkins >> System
    withSonarQubeEnv('sonarqube-server') {
      sh "${scannerHome}/bin/sonar-scanner"
    }
  }
}
stage('Unit Testing Stage') {
  steps {
    sh 'mvn surefire-report:report'
  }
}
stage('Quality Gate'){
  steps {
    script {
      timeout(time: 1, unit: 'HOURS') { // In case something goes wrong, pipeline will be killed after a timeout
      def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
      if (qg.status != 'OK') {
        error "Pipeline aborted due to quality gate failure: ${qg.status}"
      }
    }
  }
}
def registry = 'https://binhost.jfrog.io'
stage("Jar Publish") {
  steps {
    script {
      echo '<--------------- Started Publishing Jar --------------->'
        def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"artifactory_token"
        def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
        def uploadSpec = """{
            "files": [
              {
                "pattern": "jarstaging/(*)",
                "target": "libs-release-local/{1}",
                "flat": "false",
                "props" : "${properties}",
                "exclusions": [ "*.sha1", "*.md5"]
              }
            ]
        }"""
        def buildInfo = server.upload(uploadSpec)
        buildInfo.env.collect()
        server.publishBuildInfo(buildInfo)
        echo '<--------------- Jar Published Successfully --------------->'
    }
  }
}
}