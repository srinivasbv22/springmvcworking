node {
    
   def mvnHome
   def server =Artifactory.server 'azure_art'
    try{
        stage('Preparation') {
      git 'https://github.com/srinivasbv22/springmvcworking.git'
               
      mvnHome = tool 'Maven'
      }
       
       stage('Build') {
       sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
   }
   stage('SonarQube analysis') {
       /*withSonarQubeEnv('sonarqube1') {
                   sh '''mvn sonar:sonar \
            -Dsonar.host.url=http://40.87.47.38 \
            -Dsonar.login=8c19834e275a20c1aa5fa760bf5952d2e9d0949f '''
       } */
       sh '''mvn sonar:sonar \
		  -Dsonar.projectKey=srinivasbv22_springmvcworking \
		  -Dsonar.organization=srinivasbv22-github \
		  -Dsonar.host.url=https://sonarcloud.io \
		  -Dsonar.login=7524de46afc7cf6d46296c2f863cbbd9c7e1921e'''
    }
    /* stage("Quality Gate"){
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                   error "Pipeline aborted due to quality gate failure: ${qg.status}"
                  mail bcc: '', body:"${qg.status}", cc: '', from: '', replyTo: '', subject: 'Quality fail', to: 'srinivasbv22@gmail.com'
                 
              }
          }
      }  */
      stage('Artifactory upload') {
       def uploadSpec = """
       { "files": [ { "pattern": "/var/lib/jenkins/workspace/finalPipeline/target/*.war", "target": "srini1" } ] }""" 
            server.upload(uploadSpec) 
       
        }
        
     stage('downloading artifact') 
    { 
        def downloadSpec="""{ "files":[ { "pattern":"srini1/SpringMVCHibernate.war", "target":"/var/lib/jenkins/workspace/finalPipeline/" } ] }""" 
        server.download(downloadSpec)
    }    
    stage ('Final deploy'){
        sh 'scp  /var/lib/jenkins/workspace/finalPipeline/SpringMVCHibernate.war ubuntu@srinivas.eastus.cloudapp.azure.com:/opt/tomcat/apache-tomcat-8.5.14/webapps/'
    }
    currentBuild.result == 'SUCCESS'
   }
   catch(err)
   {
       currentBuild.result = 'FAILURE'
       mail bcc: '', body:"${err}", cc: '', from: '', replyTo: '', subject: 'job failed', to: 'srinivasbv22@gmail.com'
   
    stage('JIRA Issue') {
        withEnv(['JIRA_SITE=Jira1']) {
    		def testIssue = [fields: [ project: [key: 'DIS'],
                                     summary: 'New JIRA Created from Jenkins.',
                                     description: 'New JIRA Created from Jenkins.',
                                     issuetype: [name: 'Bug']]]
    
    		response = jiraNewIssue issue: testIssue
    
    		echo response.successful.toString()
    		echo response.data.toString()
    		jiraComment body: 'Build sucess', issueKey: 'DIS-2'
    		
        }
        withEnv(['JIRA_SITE=Jira1']) {
            jiraAssignIssue idOrKey: 'DIS-2', userName: 'devopsguy.mayank'
        }
        
	}
   }
    stage('JIRA Transition') {
        if(currentBuild.result == 'SUCCESS'){
        withEnv(['JIRA_SITE=Jira1']) {
            
          def transitionInput =
          [
              transition: [
                  id: '31'
              ]
          ]
    
          jiraTransitionIssue idOrKey: 'DIS-2', input: transitionInput
        }
        }
	}
    
}
