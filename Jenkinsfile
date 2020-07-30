pipeline {
  agent none
  stages {
    stage("build & SonarQube analysis") {
      agent any
      steps {
        withSonarQubeEnv('SonarQube') {
          sh 'mvn clean package sonar:sonar'
        }
      }
    }
    stage ("SonarQube analysis") { 
      steps { 
        script {
          def qualitygate = waitForQualityGate() 
          if (qualitygate.status != "OK") { 
            error "Pipeline aborted due to quality gate coverage failure: ${qualitygate.status}" 
          }         
        }
      } 
    }
    stage('Print ENV') {
      agent any
      steps {
        sh 'echo "Saving logs to a new file in ${JENKINS_HOME}/LOGS folder..."'
        sh 'cat ${JENKINS_HOME}/jobs/SonarQubeDemo/branches/${GIT_BRANCH}/builds/${BUILD_NUMBER}/log >> ${BUILD_TAG}.txt'
      }
    }
    stage('Upload to AWS') {
      agent any
      steps {
          sh 'pwd'
          withAWS(region:'us-east-1',credentials:'aws-secrets') {
            sh 'echo "Uploading content with AWS creds"'
            s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file: "${env.BUILD_TAG}.txt" , bucket:'sksingh-jenkins-786')
          }
      }
    }
  }
}
