node('docker') {
	stage('Poll') {
		checkout scm
	}
      //	stage('Build & Unit test'){
     //		sh 'mvn clean verify -DskipITs=true';
   //  		junit '**/target/surefire-reports/TEST-*.xml' 
  //    		archive 'target/*.jar'
   //	}
 
  //	stage('Static Code Analysis'){
    //		sh 'mvn clean verify sonar:sonar -Dsonar.projectName=Esafe-Project -Dsonar.projectKey=Esafe-Project -Dsonar.projectVersion=$BUILD_NUMBER';
//	}
//	stage ('Integration Test'){
  //  		sh 'mvn clean verify -Dsurefire.skip=true';
//		junit '**/target/failsafe-reports/TEST-*.xml'
 //     		archive 'target/*.jar'
//	} 
//	stage ('Publish'){
 //   		def server = Artifactory.server 'Default Artifactory Server'
  //  		def uploadSpec = """{
  //  		"files": [
 //   		{
  //   		"pattern": "target/Esafe-0.0.1.war",
  //   		"target": "Esafe-Project/${BUILD_NUMBER}/",
//	 	"props": "Integration-Tested=Yes;Performance-Tested=No"
  // 		}
 //          	]
//		}"""
//		server.upload(uploadSpec)
           //   }
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
