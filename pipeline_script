
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
       withSonarQubeEnv('sonarqube1') {
                   sh '''mvn sonar:sonar \
            -Dsonar.host.url=http://40.87.47.38 \
            -Dsonar.login=8c19834e275a20c1aa5fa760bf5952d2e9d0949f '''
       }
    }
    stage("Quality Gate"){
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                   error "Pipeline aborted due to quality gate failure: ${qg.status}"
                  mail bcc: '', body:"${qg.status}", cc: '', from: '', replyTo: '', subject: 'Quality fail', to: 'srinivasbv22@gmail.com'
                 
              }
          }
      }  
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

   }
   catch(err)
   {
       mail bcc: '', body:"${err}", cc: '', from: '', replyTo: '', subject: 'job failed', to: 'srinivasbv22@gmail.com'
   }
   
}
