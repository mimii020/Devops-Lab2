pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'mimii020/devops-lab2'
        DOCKER_CREDENTIALS = 'docker-hub-credentials'
    }
    
    tools {
        maven 'M3'
        jdk 'JDK17'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/mimii020/Devops-Lab2.git'
            }
        }
        
        stage('Build with Maven') {
            steps {
                sh 'mvn clean compile'
                echo 'Maven build completed successfully'
            }
        }
        
        stage('Package Application') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def version = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()
                    
                    echo "Building Docker image with version: ${version}"
                    
                    // Build the image with version tag
                    sh "docker build -t ${DOCKER_IMAGE}:${version} ."
                    
                    // ALSO create the latest tag
                    sh "docker tag ${DOCKER_IMAGE}:${version} ${DOCKER_IMAGE}:latest"
                    
                    // Verify both tags exist
                    sh """
                        echo "Verifying Docker images:"
                        docker images | grep devops-lab2
                    """
                }
            }
        }
        
        stage('Test Docker Image') {
            steps {
                script {
                    def version = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()
                    
                    echo "Testing Docker image: ${DOCKER_IMAGE}:${version}"
                    
                    // Test the container
                    sh "docker run -d --name test-container -p 8081:8080 ${DOCKER_IMAGE}:${version}"
                    
                    
                    // Clean up test container
                    sh 'docker stop test-container && docker rm test-container'
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    def version = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()
                    
                    echo "=== Starting Docker Hub Push ==="
                    echo "Version: ${version}"
                    echo "Image: ${DOCKER_IMAGE}"
                    
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo "🔐 Logging into Docker Hub..."
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            
                            echo "🚀 Pushing ${DOCKER_IMAGE}:${version}"
                            docker push ${DOCKER_IMAGE}:${version}
                            
                            echo "🚀 Pushing ${DOCKER_IMAGE}:latest"
                            docker push ${DOCKER_IMAGE}:latest
                            
                            docker logout
                            echo "🎉 All images pushed to Docker Hub successfully!"
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            emailext (
                subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: "The pipeline completed successfully. Docker image pushed to Docker Hub.",
                to: "imen.abidi@insat.ucar.tn"
            )
        }
        failure {
            echo 'Pipeline failed!'
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: "The pipeline failed. Please check the logs at: ${env.BUILD_URL}",
                to: "imen.abidi@insat.ucar.tn"
            )
        }
        always {
            echo 'Pipeline execution completed. Cleaning up workspace...'
            // Clean up any running containers
            sh 'docker stop test-container || true'
            sh 'docker rm test-container || true'
            cleanWs()
        }
    }
}