def registry = 'https://valaxy25.jfrog.io'
def imageName = 'valaxy05.jfrog.io/valaxy-docker-local/ttrend'
def version = '2.1.4'

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
        stage("Build") {
            steps {
                echo "----------- Build started ----------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "----------- Build completed ----------"
            }
        }
        
        stage("Unit Test") {
            steps {
                echo "----------- Unit test started ----------"
                sh 'mvn surefire-report:report'
                echo "----------- Unit test completed ----------"
            }
        }
        
        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'valaxy-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('valaxy-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        
        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        
        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.server('artfiact-cred')
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/*",
                                "target": "libs-release-local/{1}",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    server.publishBuildInfo buildInfo
                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }
        
        stage("Docker Build") {
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    try {
                        app = docker.build("${imageName}:${version}")
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    } finally {
                        echo '<--------------- Docker Build Ended --------------->'
                    }
                }
            }
        }
        
        stage("Docker Publish") {
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'
                    try {
                        docker.withRegistry("${registry}", 'artfiact-cred') {
                            app.push()
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e
                    } finally {
                        echo '<--------------- Docker Publish Ended --------------->'
                    }
                }
            }
        }
    }
}