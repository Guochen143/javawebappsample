import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=<dd3b5eb6-b3d8-406f-b912-40a38ed85880>',
        'AZURE_TENANT_ID=<1df7dbce-bfef-4448-bb5d-01022abd9a61>']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = '<jenkins-get-started-rg>'
      def webAppName = '<xiamomo143>'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '<AzureServicePrincipal>', passwordVariable: 'ee85d4a8-0b4c-4df7-ac1b-cb833f44acea', usernameVariable: 'P9Z8Q~tNoFubzQ~jrfKDqj-QXdgAF7E5~K8CncQY')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
