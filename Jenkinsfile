def secondsToApproval = "${env.seconds_to_approval}".toInteger()

node {
    try {
	
		stage('Checkout') {		
			checkout scm

			// Marca a versão curta do commit atual
			sh "git rev-parse --short HEAD > .git/commit-id"
		}
		
		stage('Setup Build Version') {
			def model = readMavenPom file: 'pom.xml'
			def commitId = readFile('.git/commit-id')
			def buildNumber = "${env.BUILD_NUMBER}"
			def releaseVersion = model.version.replace("-SNAPSHOT", "")
			def finalBuildVersion = releaseVersion + '-build.' + buildNumber + '.' + commitId
			
			echo 'POM Version found: ' + model.version
			echo 'Build version defined: ' + finalBuildVersion
			
			sh "mvn versions:set -DnewVersion=" + finalBuildVersion
			sh "mvn versions:commit"
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
                        sh "curl 'http://vivian:password@tomcat_dev:8889/manager/text/reload?path=/argentum-web'"
		}

	} catch (any) {
		currentBuild.result = 'FAILURE'
		throw any //rethrow exception to prevent the build from proceeding
	}
}
