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
                    sh 'docker-compose build'
                }
            }
        }

        stage('Tag Docker Images') {
            steps {
                script {
                    sh 'docker tag super-app-jenkins-node nimirockdev/super-app-jenkins-node:latest'
                    sh 'docker tag super-app-jenkins-php nimirockdev/super-app-jenkins-php:latest'
                }
            }
        }

        stage('Test Application') {
            steps {
                script {
                    sh 'docker-compose up -d'

                    sleep(10) // wait for the containers to start
                    
                    sh 'ls -l /opt/tomcat/.jenkins/workspace/super-app-project/php'
                    sh 'chmod -R 755 /opt/tomcat/.jenkins/workspace/super-app-project/php'

                    sh 'curl http://localhost:3000/super-app'
                    sh 'curl -f http://localhost:80'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                    }

                    sh 'docker push nimirockdev/super-app-jenkins-node:latest'
                    sh 'docker push nimirockdev/super-app-jenkins-php:latest'
                }
            }
        }
    }

    post {
        always {
            script {
                sh 'docker-compose down'
                cleanWs()
            }
        }
    }
}
