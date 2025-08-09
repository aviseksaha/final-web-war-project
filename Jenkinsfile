pipeline {
    agent any
    environment {
        PATH = "/opt/maven/bin:$PATH"
    }
    stages {
        stage('build') {
            steps {
		echo "...............build started................."
                sh 'mvn clean deploy'
		echo "...............build complete................"
            }
        }
	stage('test') {
	    steps {
	    	echo "................unit test started................"
		sh 'mvn surefire-report:report'
		echo "...............unit test completed..............."
	    }
	}
        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'Saidemy-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('saidemy-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    }
}

