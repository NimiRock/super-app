pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git credentialsId: 'github-creds', url: 'https://github.com/NimiRock/super-app.git', branch: 'main'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    // Build Docker images
                    sh 'docker-compose build'
                }
            }
        }

        stage('Tag Docker Images') {
            steps {
                script {
                    // Tag the Docker images with your Docker Hub username and a version (e.g., 'latest')
                    sh 'docker tag super-app-jenkins-node nimirockdev/super-app-jenkins-node:latest'
                    sh 'docker tag super-app-jenkins-php nimirockdev/super-app-jenkins-php:latest'
                }
            }
        }

        stage('Test Application') {
            steps {
                script {
                    // Run Docker Compose
                    sh 'docker-compose up -d'

                    // Add your test commands here
                    sleep(10) // wait for the containers to start
                    
                    sh 'ls -l /opt/tomcat/.jenkins/workspace/super-app-project/php'
                    sh 'chmod -R 755 /opt/tomcat/.jenkins/workspace/super-app-project/php'

                    // Test PHP app
                    sh 'curl http://localhost:3000/super-app'
                    sh 'curl -f http://localhost:80'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Log in to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                    }

                    // Push the tagged images
                    sh 'docker push nimirockdev/super-app-jenkins-node:latest'
                    sh 'docker push nimirockdev/super-app-jenkins-php:latest'
                }
            }
        }
    }

    post {
        always {
            script {
                // Clean up Docker containers and workspace after the pipeline run
                sh 'docker-compose down'
                cleanWs()
            }
        }
    }
}
