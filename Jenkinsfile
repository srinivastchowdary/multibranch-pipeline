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
