pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'
    }

    environment {
        TOMCAT_HOST = 'YOUR_TOMCAT_IP'
        TOMCAT_USER = 'ec2-user'
        DEPLOY_PATH = '/opt/tomcat/webapps'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/YOUR_USERNAME/YOUR_REPO.git'
            }
        }

        stage('Verify Maven') {
            steps {
                sh '''
                mvn --version
                java -version
                '''
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.war'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sshagent(credentials: ['tomcat-ssh']) {
                    sh """
                    scp -o StrictHostKeyChecking=no target/*.war \
                    ${TOMCAT_USER}@${TOMCAT_HOST}:${DEPLOY_PATH}/
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: ['tomcat-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} '
                    ls -lh ${DEPLOY_PATH}
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Application deployed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }

        always {
            cleanWs()
        }
    }
}
