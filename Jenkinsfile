pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_IMAGE', defaultValue: 'simonzheng050428/teedy', description: 'Docker Hub repository, for example username/teedy')
    }

    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub_credentials'
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
        IMAGE_NAME = "${params.DOCKER_IMAGE}"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        CONTAINER_PREFIX = 'teedy-container'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build WAR') {
            steps {
                script {
                    runMaven('-B -DskipTests clean package')
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.IMAGE_NAME}:${env.DOCKER_TAG}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_HUB_CREDENTIALS, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        dockerLogin()
                        runDocker("docker push ${env.IMAGE_NAME}:${env.DOCKER_TAG}")
                        runDocker("docker tag ${env.IMAGE_NAME}:${env.DOCKER_TAG} ${env.IMAGE_NAME}:latest")
                        runDocker("docker push ${env.IMAGE_NAME}:latest")
                    }
                }
            }
        }

        stage('Run Three Containers') {
            steps {
                script {
                    [8082, 8083, 8084].each { port ->
                        def name = "${env.CONTAINER_PREFIX}-${port}"
                        runDocker("docker rm -f ${name} || true")
                        runDocker("docker run -d --name ${name} -p ${port}:8080 ${env.IMAGE_NAME}:${env.DOCKER_TAG}")
                    }
                    runDocker("docker ps --filter \"name=${env.CONTAINER_PREFIX}\"")
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'docs-web/target/*.war', fingerprint: true, allowEmptyArchive: true
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

def runDocker(String command) {
    if (isUnix()) {
        sh command
    } else {
        bat command
    }
}

def dockerLogin() {
    if (isUnix()) {
        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
    } else {
        bat 'echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin'
    }
}
