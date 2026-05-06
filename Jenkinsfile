pipeline {
    agent any
    stages {
        stage('Clean') {
            steps {
                script {
                    runMaven('clean')
                }
            }
        }
        stage('Compile') {
            steps {
                script {
                    runMaven('compile')
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    runMaven('test -Dmaven.test.failure.ignore=true')
                }
            }
        }
        stage('PMD') {
            steps {
                script {
                    runMaven('pmd:pmd')
                }
            }
        }
        stage('JaCoCo') {
            steps {
                script {
                    runMaven('jacoco:report')
                }
            }
        }
        stage('Javadoc') {
            steps {
                script {
                    runMaven('javadoc:javadoc')
                }
            }
        }
        stage('Site') {
            steps {
                script {
                    runMaven('site')
                }
            }
        }
        stage('Package') {
            steps {
                script {
                    runMaven('package -DskipTests')
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: '**/target/site/**/*.*', fingerprint: true, allowEmptyArchive: true
            archiveArtifacts artifacts: '**/target/**/*.jar', fingerprint: true, allowEmptyArchive: true
            archiveArtifacts artifacts: '**/target/**/*.war', fingerprint: true, allowEmptyArchive: true
            junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
        }
    }
}

def runMaven(String args) {
    if (isUnix()) {
        sh "mvn ${args}"
    } else {
        bat "mvn ${args}"
    }
}
