import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
  return null
}

pipeline {
  agent any
  
  environment {
    AZURE_SUBSCRIPTION_ID = '<69d958df-11de-4ec7-9cec-98616f7ab281>'
    AZURE_TENANT_ID = '<91ddffdf-038f-4435-9230-c76a677d6a2a>'
    AZURE_CLIENT_ID = credentials('0dab7181-b2fd-43c7-b49c-92f4322741b2')
    AZURE_CLIENT_SECRET = credentials('ocF8Q~I96NEkDTqXV1ye1p4J_llY3nTuiRFu0b10')
    RESOURCE_GROUP = '<Workshop1>'
    WEBAPP_NAME = '<app_name>'
  }
  
  stages {
    stage('init') {
      steps {
        checkout scm
      }
    }
  
    stage('build') {
      steps {
        sh 'mvn clean package'
      }
    }
  
    stage('deploy') {
      steps {
        script {
          // Login to Azure
          withCredentials([usernamePassword(credentialsId: AZURE_CLIENT_ID, passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
            sh '''
              az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
              az account set -s $AZURE_SUBSCRIPTION_ID
            '''
          }
          
          // Get publish settings
          def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $RESOURCE_GROUP -n $WEBAPP_NAME --query '[].{publishMethod: publishMethod, publishUrl: publishUrl, userName: userName, userPWD: userPWD}' --output json", returnStdout: true
          def ftpProfile = getFtpPublishProfile(pubProfilesJson)
          
          if (ftpProfile) {
            // Upload package
            sh "curl -T target/calculator-1.0.war ${ftpProfile.url}/webapps/ROOT.war -u '${ftpProfile.username}:${ftpProfile.password}'"
          } else {
            error "FTP publish profile not found"
          }
          
          // Logout from Azure
          sh 'az logout'
        }
      }
    }
  }
}
