pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    
    environment {
        PATH = "/opt/apache-maven-3.9.3/bin:$PATH"
        SONAR_PROJECT_KEY = 'ttrend01'
        SONAR_ORGANIZATION = 'valaxy01r-key'
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
            }
            steps {
                script {
                    withSonarQubeEnv('valaxy-sonarqube-server') {
                        sh "${scannerHome}/bin/sonar-scanner -X -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.organization=${SONAR_ORGANIZATION} -Dsonar.java.binaries=${SONAR_JAVA_BINARIES} -Dsonar.exclusions=${SONAR_EXCLUSIONS}"
                    }
                }
            }
        }
    }
}

