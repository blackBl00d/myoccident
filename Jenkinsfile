pipeline {
    agent any
    
    environment {
        DOCKER_COMPOSE_PROD = 'docker-compose.prod.yml'
        APP_NAME = 'your-app'
        DEPLOY_DIR = '/opt/deployments/${APP_NAME}'  // Deployment directory on server
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build with build number as tag
                    sh "docker-compose -f ${DOCKER_COMPOSE_PROD} build"
                }
            }
        }
        
        stage('Prepare Deployment') {
            steps {
                script {
                    // Create deployment directory if it doesn't exist
                    sh """
                        mkdir -p ${DEPLOY_DIR}
                        cp ${DOCKER_COMPOSE_PROD} ${DEPLOY_DIR}/docker-compose.yml
                    """
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    dir("${DEPLOY_DIR}") {
                        // Stop existing container (if any)
                        sh "docker-compose down || true"
                        
                        // Start new container
                        sh """
                            BUILD_NUMBER=${BUILD_NUMBER} docker-compose up -d
                            docker image prune -f
                        """
                    }
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    // Wait for application to be ready
                    sh 'sleep 10'
                    
                    // Basic health check
                    sh 'curl -f http://localhost:3002 || exit 1'
                }
            }
        }
    }
    
    post {
        failure {
            script {
                // Rollback on failure
                dir("${DEPLOY_DIR}") {
                    sh """
                        docker-compose down || true
                        docker-compose up -d --no-build
                    """
                }
            }
        }
        always {
            // Cleanup old images
            sh 'docker image prune -f'
        }
    }
}
