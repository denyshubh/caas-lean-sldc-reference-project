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
          sh 'mvn clean package sonar:sonar'
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
    stage('Saving Logs') {
      agent any
      steps {
          sh 'printenv'
          sh 'echo "Saving logs to a new file in ${JENKINS_HOME}/LOGS folder..."'
          sh 'cat ${JENKINS_HOME}/jobs/${PROJECT_NAME}/branches/${GIT_BRANCH}/builds/${BUILD_NUMBER}/log >> ${BUILD_TAG}.txt'
          sh 'pwd'
          sh 'python3 /home/ubuntu/generate.py ${BUILD_TAG}.txt'
          sh 'mkdir ${WORKSPACE}/target/surefire-reports/unit-test'
          sh 'cp ${WORKSPACE}/target/surefire-reports/*.txt ${WORKSPACE}/target/surefire-reports/unit-test'
          sh 'ls ${WORKSPACE}/target/surefire-reports/unit-test'
      }
    }
    stage('Upload to AWS') {
      agent any
      steps {
        sh 'pwd'
        script {
          def date = new Date().format("yyyy-MM-dd", TimeZone.getTimeZone('UTC'))
          withAWS(region:'us-east-1',credentials:'aws-secrets') {
            sh 'echo "Uploading content with AWS creds"'
            s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file: "${env.BUILD_TAG}.txt" , bucket:'sksingh-jenkins-786', path: "SonarLogs/${date}/${env.BUILD_TAG}.txt")
            s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file: "Sonar-analysis-${env.BUILD_TAG}.txt" , bucket:'sksingh-jenkins-786', path: "SonarLogs/${date}/Sonar-analysis-${env.BUILD_TAG}.txt")
            s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file: "${env.WORKSPACE}/target/surefire-reports/unit-test" , bucket:'sksingh-jenkins-786', path: "SonarLogs/${date}/Unit-Test-${env.BUILD_TAG}")
          }
        }
      }
    }
  }
}