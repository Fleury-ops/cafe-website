pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'cafe-website'
        DOCKER_TAG = "${BUILD_NUMBER}"
        GITHUB_REPO = 'Fleury-ops/cafe-website'  
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                git branch: 'main',
                    url: "https://github.com/${GITHUB_REPO}.git"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                script {
                    // Run container to test
                    sh "docker run -d -p 8081:80 --name test-container ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "sleep 5"
                    sh "curl -f http://localhost:8081 || exit 1"
                    sh "docker stop test-container"
                    sh "docker rm test-container"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                script {
                    // Stop old container if exists
                    sh """
                        docker stop cafe-website-prod || true
                        docker rm cafe-website-prod || true
                    """
                    
                    // Run new container
                    sh """
                        docker run -d \
                        -p 80:80 \
                        --name cafe-website-prod \
                        --restart unless-stopped \
                        ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
        
        stage('Clean Up') {
            steps {
                echo 'Cleaning up old images...'
                script {
                    sh """
                        docker image prune -f
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully! ✅'
            echo 'Website is now running at http://localhost'
        }
        failure {
            echo 'Pipeline failed! ❌'
            echo 'Check the console output for errors'
        }
    }
}