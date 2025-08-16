pipeline {
    agent any
    environment {
        PATH = "/opt/maven/bin:$PATH"
        AWS_REGION = "ap-south-1"
        DOMAIN_NAME = "final-web-war-project"
        REPO_NAME = "final-web-war-project"
        DOMAIN_OWNER = "509399629743"
    }
    stages {
        stage('Build') {
            steps {
                echo "...............build started................."
                sh 'mvn clean install'
                echo "...............build complete................"
            }
        }
        stage('Test') {
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
        stage('Upload WAR to AWS CodeArtifact') {
			steps {
				withAWS(credentials: 'aws-cred', region: "${AWS_REGION}") {
					// Fetch CodeArtifact repository endpoint
					sh """
						REPO_ENDPOINT=\$(aws codeartifact get-repository-endpoint \
							--domain ${DOMAIN_NAME} \
							--domain-owner ${DOMAIN_OWNER} \
							--repository ${REPO_NAME} \
							--format maven --query repositoryEndpoint --output text)
					"""

					// Fetch CodeArtifact authorization token
					sh """
						AUTH_TOKEN=\$(aws codeartifact get-authorization-token \
							--domain ${DOMAIN_NAME} \
							--domain-owner ${DOMAIN_OWNER} \
							--query authorizationToken --output text)
					"""

					// Create Maven settings.xml with CodeArtifact credentials
					sh """
						mkdir -p ~/.m2
						cat > ~/.m2/settings.xml << EOF
						<settings>
							<servers>
								<server>
									<id>codeartifact</id>
									<username>aws</username>
									<password>\${AUTH_TOKEN}</password>
								</server>
							</servers>
						</settings>
						EOF
					"""

					// Deploy WAR to CodeArtifact using Maven
					sh """
						mvn deploy:deploy-file \
							-DgroupId=com.example \
							-DartifactId=myapp \
							-Dversion=1.0.0 \
							-Dpackaging=war \
							-Dfile=target/myapp.war \
							-DrepositoryId=codeartifact \
							-Durl=\${REPO_ENDPOINT}
					"""
				}
			}
		}
    }
}

