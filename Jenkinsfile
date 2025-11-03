#!groovy

pipeline {
    agent any
    environment {
        HOME = "${env.WORKSPACE}"
    }

    stages {

        stage('Docker environment') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps {
                sh"""
                export PIP_DISABLE_PIP_VERSION_CHECK=1
                export PATH="$HOME/.local/bin:${PATH}"
                pip install -r requirements.txt
                pip install -r requirements-test.txt
                """
            }
        }

        stage('Unit test')
        
        stage('Deliver') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-id',
                    passwordVariable: 'passwd', 
                    usernameVariable: 'username')]) {
                        sh"""
                        printenv
                        docker build -t ${username}/${JOB_BASE_NAME} .
                        docker login -u ${username} -p ${passwd}
                        docker push ${username}/${JOB_BASE_NAME}
                        """
               }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-id',
                    passwordVariable: 'passwd', 
                    usernameVariable: 'username')]) {
                    sshagent(credentials: ['cluster-crendentials']) {
                        sh"""
                        ssh -o StrictHostKeyChecking=no redhat@172.31.33.7 docker rm -f ${JOB_BASE_NAME} || true
                        ssh -o StrictHostKeyChecking=no redhat@172.31.33.7 docker run -d --name ${JOB_BASE_NAME} -p 8080:9046 ${username}/${JOB_BASE_NAME}
                        """
                    }
                }
            }
        }
    }
}
