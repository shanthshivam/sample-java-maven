# sample-java-maven
DevOps demo


// multistage
pipeline {
    agent any
    tools { 
        maven 'M3' 
    }
        stages {
            stage('Source') {
                steps {
                    git url: 'https://github.com/shanthshivam/sample-java-maven.git'
                }
            }
            stage('Build') {
                steps {
                    script {
                        def mvnHome = tool 'M3'
                        sh "mvn -B verify"
                    }
                }
            }
            stage('SonarQube Analysis') {
                steps {
                    script {
                        def mvnHome = tool 'M3'
                        withSonarQubeEnv() {
                            sh "mvn clean verify sonar:sonar -Dsonar.projectKey=java-maven"
                        }
                    }
                }
            }
            stage('Test') {
                steps {
                    step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
                }
            }
            stage('Packaging') {
                steps {
                    step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
                }
            }
        }
}
