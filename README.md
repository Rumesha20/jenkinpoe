# jenkinpoe
setup jenkins job that picks upan application from github repository build it and run it and dockerize the application

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'myapp:latest'
        DOCKER_REGISTRY = 'mydockerhubaccount'
        GITHUB_REPO = 'https://github.com/yourusername/yourrepository.git'
    }

    stages {
        stage('Clone GitHub Repository') {
            steps {
                git branch: 'main', url: "${GITHUB_REPO}"
            }
        }

        stage('Build Application') {
            steps {
                script {
                    // If you are building a Java app with Maven, for example:
                    sh 'mvn clean install'
                    // Or for Node.js
                    // sh 'npm install'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image using Dockerfile
                    sh '''
                        docker build -t ${DOCKER_IMAGE} .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // If you want to push the Docker image to Docker Hub
                    docker.withRegistry("https://registry.hub.docker.com", "dockerhub-credentials-id") {
                        sh "docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    // Run the Docker container
                    sh '''
                        docker run -d -p 8080:8080 ${DOCKER_IMAGE}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Build and Dockerization successful!'
        }
        failure {
            echo 'Build or Dockerization failed!'
        }
    }
}
