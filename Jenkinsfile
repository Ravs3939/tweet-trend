pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    
    environment {
        PATH = "/opt/apache-maven-3.9.3/bin:$PATH"
    }
    
    stages {
        stage("build") {
            steps {
                sh 'mvn clean deploy'
            }
        }
        
        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'valaxy-sonar-scanner'
                SONAR_PROJECT_KEY = 'rvzpro7_twittertrend'
                SONAR_ORGANIZATION = 'valaxy01'
                SONAR_PROJECT_NAME = 'twittertrend'
            }
            steps {
                script {
                    withSonarQubeEnv('valaxy-sonarqube-server') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.organization=${SONAR_ORGANIZATION} -Dsonar.projectName=${SONAR_PROJECT_NAME}"
                    }
                }
            }
        }
    }
}
