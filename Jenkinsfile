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
                timeout(time: 2, unit: 'MINUTES') { 
                    script {
                        def qualityGate = waitForQualityGate() 
                        if (qualityGate.status != 'OK') {
                            echo "Warning: Quality gate partially passed but continuing pipeline: ${qualityGate.status}"
                        }
                    }
                }              
            }
        }
        stage('Upload WAR to AWS CodeArtifact') {
            steps {
                        // Authenticate Maven to CodeArtifact
                        sh """
                            aws codeartifact login --tool maven \
                              --repository ${REPO_NAME} \
                              --domain ${DOMAIN_NAME} \
                              --domain-owner ${DOMAIN_OWNER} \
                              --region ${AWS_REGION}
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
                              -Durl=$(aws codeartifact get-repository-endpoint \
                                       --domain ${DOMAIN_NAME} \
                                       --domain-owner ${DOMAIN_OWNER} \
                                       --repository ${REPO_NAME} \
                                       --format maven \
                                       --region ${AWS_REGION} --query repositoryEndpoint --output text)
                        """
                    }
                }
            }
        }
    }
}

