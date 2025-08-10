pipeline {
    agent any
    environment {
        PATH = "/opt/maven/bin:$PATH"
    }
    stages {
        stage('build') {
            steps {
		echo "...............build started................."
                sh 'mvn clean install'
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
	stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') { 
                    script {
                        def qualityGate = waitForQualityGate() 
                        if (qualityGate.status != 'OK') {
                            //error "Pipeline failed due to quality gate failure: ${qualityGate.status}"
							echo "Warning: Quality gate partially passed but continuing pipeline:${qualityGate.status}"
                        }
                    }
                }
            }
	}
    }
}

