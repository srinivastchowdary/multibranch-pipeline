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
