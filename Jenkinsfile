pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        GIT_REPO_URL = 'https://github.com/kustav123/demo.git'
        GIT_BRANCH = 'master'
        DOCKER_IMAGE = 'repo.kustav.co.in/app/'
    }

    stages {
        stage('Clone Repo') {
            steps {
                    git branch: "${env.GIT_BRANCH}", url: "${env.GIT_REPO_URL}"
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    def commitId = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def fullTag = "${DOCKER_IMAGE}:${commitId}"

                    withCredentials([usernamePassword(
                        credentialsId: 'Docker', 
                        usernameVariable: 'DOCKER_USER', 
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo "$DOCKER_PASS" | docker login repo.kustav.co.in -u "$DOCKER_USER" --password-stdin
                            docker build -t ${fullTag} .
                            docker push ${fullTag}
                            docker rmi ${fullTag} || true
                            docker logout
                        """
                    }
                }
            }
        }

        stage('Start Application with Docker Compose') {
            steps {
                script {
                    def commitId = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def fullTag = "${DOCKER_IMAGE}:${commitId}"
                    def processorTag = "${PROCESSOR_IMAGE}:${commitId}"
                    sh """
                        export fullTag=${fullTag}
                        docker compose up -d
                    """
                }
            }
        }
    }
}
