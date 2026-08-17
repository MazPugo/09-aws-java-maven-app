#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMSource',
     remote: 'https://github.com/MazPugo/jenkins-shared-library.git',
     credentialsId: 'github-credentials'
    ]
)

pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        IMAGE_NAME = 'mazpugo/demo-app:java-maven-1.0'
        EC2_HOST   = 'ec2-user@3.9.23.167'
    }

    stages {

        stage('build app') {
            steps {
                echo 'building application jar...'
                buildJar()
            }
        }
        stage('build image') {
            steps {
                script {
                    echo 'building the docker image...'
                    buildimage(env.IMAGE_NAME)
                    DockerLogin()
                    DockerPush(env.IMAGE_NAME)
                }
            }
        }

        stage('deploy') {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def shellCmd = "bash ./server-cmds.sh"

                    sshagent(['ec2-server-key']) {
                        sh "scp server-cmds.sh ec2-user@3.9.23.167:/home/ec2-user"
                        sh "scp docker-compose.yaml ec2-user@3.9.23.167:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@3.9.23.167 ${shellCmd}"
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed — check the stage logs above.'
        }
        always {
            cleanWs()
        }
    }
}