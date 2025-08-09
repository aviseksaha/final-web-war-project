pipeline {
    agent any
    environment {                           // (If you mentioned in tools then it's optional)
        PATH = "/opt/maven/bin:$PATH"
    }
    stages {
        stage('stage 1') {                   // creates new stage "stage1"
            steps {                          // defines the steps
                echo "Hello world!"          // print to console
            }
        }
        stage('git clone') {
            steps {
                git url: 'https://github.com/aviseksaha/final-web-war-project.git', branch: 'main'   // Java code
            }
        }
        stage('build') {
            steps {
                sh 'mvn clean install'
            }
        }
    }
}

