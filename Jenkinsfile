def project="${JOB_NAME}".split('/')[0]
pipeline {
  agent none
  environment {
    PROJECT_NAME= "${project}"
  }
  stages {
    stage("build & SonarQube analysis") {
      agent any
      steps {
        withSonarQubeEnv('SonarQube') {
          sh '/opt/maven/bin/mvn clean package -X sonar:sonar'
        }
      }
    }
    stage ("SonarQube analysis") { 
      agent none
      steps { 
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          script {
            timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
            def qualitygate = waitForQualityGate() 
            if (qualitygate.status != "OK") { 
              error "Pipeline aborted due to quality gate coverage failure: ${qualitygate.status}" 
              }
            }
          }
        } 	  
      }
    }
  }
}
