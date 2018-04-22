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

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
		sh '''
		    echo "Sonar finished"
                '''
            }
        }

        stage('Deploy DEV') {				
            steps {
                timeout(time: secondsToApproval, unit: 'SECONDS') {
                    input "Deploy to DEV?"
                }
            },				
            steps {
                sh '''
                    cp /jenkins/workspace/argentum-web-build/target/argentum-web.war /tomcat/DEV
                    curl 'http://vivian:password@tomcat_dev:8080/manager/text/reload?path=/argentum-web'
                    echo "Deploy to DEV finished"
                '''
            }
        }

        stage('Deploy TI') {				
            steps {
                timeout(time: secondsToApproval, unit: 'SECONDS') {
                    input "Deploy to TI?"
                }
            },				
            steps {
                sh '''
                    cp /jenkins/workspace/argentum-web-build/target/argentum-web.war /tomcat/TI
                    curl 'http://vivian:password@tomcat_ti:8889/manager/text/reload?path=/argentum-web'
                    echo "Deploy to TI finished"
                '''
            }
        }

    } catch (any) {
        currentBuild.result = 'FAILURE'
        throw any //rethrow exception to prevent the build from proceeding
    }
}
