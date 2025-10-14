pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'mimii020/devops-lab2'
        DOCKER_IMAGE = 'devops-lab2'
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
                    def timestamp = sh(script: 'date +%Y%m%d%H%M%S', returnStdout: true).trim()
                    
                    sh "docker build -t ${DOCKER_IMAGE}:${version} ."
                }
            }
        }
        
        stage('Test Docker Image') {
            steps {
                script {
                    def version = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()
                    
                    docker.image("${DOCKER_IMAGE}:${version}").withRun('-p 8081:8080') { container ->
                        sh 'sleep 30' // Wait for application to start
                        sh 'curl -f http://localhost:8080/health || exit 1'
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
    steps {
        script {
            def version = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()
            
            // Build the image with version tag
            sh "docker build -t mimii020/devops-lab2:${version} ."
            
            // ALSO create the latest tag
            sh "docker tag mimii020/devops-lab2:${version} mimii020/devops-lab2:latest"
            
            // Verify both tags exist
            sh """
                echo "Verifying Docker images:"
                docker images | grep devops-lab2
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
            cleanWs()
        }
    }

