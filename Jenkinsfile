import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=a5cdc8eb-472c-4b8b-a3a8-70c2ba30f7bb',
        'AZURE_TENANT_ID=8a09f2d7-8415-4296-92b2-80bb4666c5fc']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = '<resource_group>'
      def webAppName = 'xavi-0ne'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '1ed09f7d-a893-4649-bae1-51f137b2bd11', passwordVariable: '~A08u-4-RW0i0B_j9~fUA81q-K4ymhfv3A', usernameVariable: 'b8ac41e8-342b-41c9-81a6-e4eee7448d0a')]) {
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
