node('docker') {
stage('Poll') {
    checkout scm
}
stage('Build'){
   	  sh 'mvn clean verify -DskipITs=true';
}
stage('Static Code Analysis'){
    sh 'mvn clean verify sonar:sonar';
}
stage ('Integration Test'){
    sh 'mvn clean verify -Dsurefire.skip=true';
}
stage ('Publish'){
    def server = Artifactory.server 'Default Artifactory Server'
    def uploadSpec = """{
    "files": [
    {
     "pattern": "target/hello-0.0.1.war",
     "target": "helloworld-greeting-project/${BUILD_NUMBER}/",
	 "props": "Integration-Tested=Yes;Performance-Tested=No"
   }
           ]
}"""
server.upload(uploadSpec)
}
}
