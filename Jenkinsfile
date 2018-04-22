def secondsToApproval = "${env.seconds_to_approval}".toInteger()

node {
    try {
	
		stage('Checkout') {		
			checkout scm

			// Marca a versão curta do commit atual
			sh "git rev-parse --short HEAD > .git/commit-id"
		}

		stage('Build') {
			sh "mvn clean install"
		}

		stage('Deploy approval DEV') {
			timeout(time: secondsToApproval, unit: 'SECONDS') {
				input "Deploy to dev?"
			}
		}

		stage('Deploy to Dev') {
			sh "cp /jenkins/workspace/argentum-web-build/target/argentum-web.war /tomcat/DEV"
                        sh "curl 'http://vivian:password@tomcat_dev:8080/manager/text/reload?path=/argentum-web'"
		}

		stage('Deploy approval TI') {
			timeout(time: secondsToApproval, unit: 'SECONDS') {
				input "Deploy to TI?"
			}
		}

		stage('Deploy to QA') {
			sh "cp /jenkins/workspace/argentum-web-build/target/argentum-web.war /tomcat/TI"
                        sh "curl 'http://vivian:password@tomcat_ti:8889/manager/text/reload?path=/argentum-web'"
		}

	} catch (any) {
		currentBuild.result = 'FAILURE'
		throw any //rethrow exception to prevent the build from proceeding
	}
}
