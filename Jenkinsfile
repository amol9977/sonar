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
}