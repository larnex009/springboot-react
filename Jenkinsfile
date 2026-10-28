pipeline {
    agent any

    tools {
        jdk 'JDK17'
        maven 'Maven3'
    }

    environment {
        SERVICE_NAME = 'simulation-service'
        DEPLOY_DIR = '/opt/finedge/services'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Package') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh './mvnw sonar:sonar'
                }
            }
        }

        stage('Deploy to VM via SSH') {
            steps {
                sshagent(['jenkins-ssh-key']) {
                    script {
                        sh "ssh -o StrictHostKeyChecking=no root@192.168.40.128 'mkdir -p ${DEPLOY_DIR}'"
                        sh "scp -o StrictHostKeyChecking=no target/*.jar root@192.168.40.128:${DEPLOY_DIR}/${SERVICE_NAME}.jar"
                        sh "ssh -o StrictHostKeyChecking=no root@192.168.40.128 'supervisorctl restart ${SERVICE_NAME}'"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! Microservice deployed and restarted via Supervisor."
        }
        failure {
            echo "Pipeline failed. Please check the logs above."
        }
    }
}
