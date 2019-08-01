node('docker') {
	stage('Poll') {
		checkout scm
	}
	stage('Build & Unit test'){
		sh 'mvn clean verify -DskipITs=true';
      		junit '**/target/surefire-reports/TEST-*.xml'
      		archive 'target/*.jar'
   	}
	stage('Static Code Analysis'){
    		sh 'mvn clean verify sonar:sonar -Dsonar.projectName=Esafe-Project -Dsonar.projectKey=Esafe-Project -Dsonar.projectVersion=$BUILD_NUMBER';
	}
	stage ('Integration Test'){
    		sh 'mvn clean verify -Dsurefire.skip=true';
		junit '**/target/failsafe-reports/TEST-*.xml'
      		archive 'target/*.jar'
	}
	stage ('Publish'){
    		def server = Artifactory.server 'Default Artifactory Server'
    		def uploadSpec = """{
    		"files": [
    		{
     		"pattern": "target/Esafe-0.0.1.war",
     		"target": "Esafe-Project/${BUILD_NUMBER}/",
	 	"props": "Integration-Tested=Yes;Performance-Tested=No"
   		}
           	]
		}"""
		server.upload(uploadSpec)
	}
	stash includes: 'target/Esafe-0.0.1.war,src/pt/Hello_World_Test_Plan.jmx', name: 'binary'
}
node('docker_pt') {
	stage ('Start Tomcat'){
    		sh '''cd /home/jenkins/tomcat/bin
    		./startup.sh''';
	}
	stage ('Deploy '){
    		unstash 'binary'
    		sh 'cp target/Esafe-0.0.1.war /home/jenkins/tomcat/webapps/';
	}
	stage ('Performance Testing'){
    		sh '''cd /opt/jmeter/bin/
    		./jmeter.sh -n -t $WORKSPACE/src/pt/Hello_World_Test_Plan.jmx -l $WORKSPACE/test_report.jtl''';
		step([$class: 'ArtifactArchiver', artifacts: '**/*.jtl'])
	}
	stage ('Promote build in Artifactory'){
    		withCredentials([usernameColonPassword(credentialsId: 'artifactory-account', variable: 'credentials')]) {
    			sh 'curl -u${credentials} -X PUT "http://192.168.0.203:8081/artifactory/api/storage/Esafe-Project/${BUILD_NUMBER}/Esafe-0.0.1.war?properties=Performance-Tested=Yes"';
		}
	}
  }
node {
	   stage('Deploy to ansiblesaerver'){
             def server = Artifactory.server 'Default Artifactory Server'
             def downloadSpec = """{
             "files": [
              {
              "pattern": "Esafe-Project/$BUILD_NUMBER/*.war",
              "target": "/opt/ansible/",
              "props": "Performance-Tested=Yes;Integration-Tested=Yes",
              "flat": "true"
               }
               ]
               }"""
               server.download(downloadSpec)
               }
	stage('Run Playbook'){
		      sh 'ansible-playbook /opt/ansible/copywarfile.yml'
	}
	stage('Email Notification'){
               mail bcc: '', body: 'Welcome to jenkins notification alert', 
               cc: 'mohamed.sadiqh@gmail.com', from: '', replyTo: '', subject: 'Jenkins job', to: 'vasucena145@gmail.com'
            }
        stage('Slack Notification'){
                slackSend baseUrl: 'https://esafeworkspace.slack.com/services/hooks/jenkins-ci/', 
	        channel: '#pipeline', color: 'good', failOnError: true, message: 'Welcome to Jenkins, Slack!', 
	        teamDomain: 'esafe build notification', tokenCredentialId: 'jenkins-slack-notification'
            }
        stage('Attachment Log'){
                 emailext attachLog: true, body: '${currentBuild.result}: ${BUILD_URL}', 
                 compressLog: true, replyTo: 'mohamed.sadiqh@gmail.com', 
                 subject: 'Build Notification: ${JOB_NAME}-Build# ${BUILD_NUMBER} ${currentBuild.result}', to: 'vasucena145@gmail.com'
            } 
 }
