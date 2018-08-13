node ('master') {

	environment {
		JAVA_HOME	= '/devops_tools/java/jdk'
		JRE_HOME    = '/devops_tools/java/jre'
	}

	//sh 'printenv'

	try
	{
		stage 'Checkout'
		checkout scm
	} 
	catch (Exception e) {
		//notifyBuild("Checkout Failure")
		throw e
	}

	withMaven(maven: 'M3') {

		//stage('Build') {
		//	sh 'mvn clean install -DskipITs=true -Dmaven.test.skip=true';
		//}
   
        stage('SonarQube analysis') {
          	withSonarQubeEnv('default') {
                // requires SonarQube Scanner for Maven 3.2+
                def (name, node) = "$JOB_NAME".tokenize( '/' )
                sh "mvn clean verify sonar:sonar -Dsonar.projectName=$name -Dsonar.projectKey=$name -Dsonar.projectVersion=$BUILD_NUMBER";
              	//sh "mvn sonar:sonar -Dsonar.host.url=http://10.2.2.31:9000 -Dsonar.login=a9085f7259e802b310095d3d4b6313042786782f -Dsonar.projectName=$name -Dsonar.projectKey=$name -Dsonar.projectVersion=$BUILD_NUMBER";
            }
        }
    }
}    

// Pre-requisites:
// SonarQube server 6.2+ (need webhook feature)
// Configure a webhook in your SonarQube server pointing to <your Jenkins instance>/sonarqube-webhook/ (info) The trailing slash is mandatory with SonarQube 6.2 and 6.3!
// Use withSonarQubeEnv step in your pipeline (so that SonarQube taskId is correctly attached to the pipeline context).

// No need to occupy a node
stage("Quality Gate"){
   timeout(time: 30, unit: 'MINUTES') { // Just in case something goes wrong, pipeline will be killed after a timeout
       def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
       if (qg.status != 'OK') {
           error "Pipeline aborted due to quality gate failure: ${qg.status}"
        }
    }
}
