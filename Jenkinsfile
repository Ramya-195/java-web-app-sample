import groovy.json.JsonSlurper

def getAcrLoginServer(def acrSettingsJson) {
  def acrSettings = new JsonSlurper().parseText(acrSettingsJson)
  return acrSettings.loginServer
}

node {
 withEnv(['AZURE_SUBSCRIPTION_ID=</subscriptions/69d958df-11de-4ec7-9cec-98616f7ab281>',
        'AZURE_TENANT_ID=<69d958df-11de-4ec7-9cec-98616f7ab281>']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def webAppResourceGroup = 'pcappservicelinuxrg'
      def webAppName = '<resource_group>'
      def acrName = '<app_name>'
      def imageName = '<registry>'
      // generate version, it's important to remove the trailing new line in git describe output
      def version = sh script: 'git describe | tr -d "\n"', returnStdout: true
      withCredentials([usernamePassword(credentialsId: '<app-demo>', passwordVariable: '4QE8Q~cFg3upWoup6aYgLmr4sSGcEKKMOVoR6at0', usernameVariable: '0dab7181-b2fd-43c7-b49c-92f4322741b2
')]) {
        // login Azure
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
         // get login server
        def acrSettingsJson = sh script: "az acr show -n $acrName", returnStdout: true
        def loginServer = getAcrLoginServer acrSettingsJson
        // login docker
        // docker.withRegistry only supports credential ID, so use native docker command to login
        // you can also use docker.withRegistry if you add a credential
        sh "docker login -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET $loginServer"
        // build image
        def imageWithTag = "$loginServer/$imageName:$version"
        def image = docker.build imageWithTag
        // push image
        image.push()
        // update web app docker settings
        sh "az webapp config container set -g $webAppResourceGroup -n $webAppName -c $imageWithTag -r http://$loginServer -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET"
        // log out
        sh 'az logout'
        sh "docker logout $loginServer"
      }
    }
  }
}
