pipeline {
    environment {
        imagename = "smartuser/trainschedule"
        registryCredentials = 'smart-dockerhub-credentials'
        dockerImage = ''
    }
    agent any
    stages {
        stage('Git clone') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/smart-cn/cicd-pipeline-train-schedule-jenkins.git'
            }
        }
        stage('Build Docker image') {
            steps{
                script {
                    dockerImage = docker.build imagename
                }
            }
        }
        stage('Push Docker image to Registry') {
            steps{
                script {
                    docker.withRegistry( '', registryCredentials ) {
                    dockerImage.push("$BUILD_NUMBER")
                    dockerImage.push('latest')
                    }     
                }
            }
        }
        stage('Remove unused Docker images and containers') {
            steps{
                sh "docker rmi $imagename:$BUILD_NUMBER"
                sh "docker rmi $imagename:latest"
                sh "docker container prune -f"
                sh "docker image prune -f"
            }
        }
        stage('Deploy new Docker image to server'){
            steps([$class: 'BapSshPromotionPublisherPlugin']) {
                input 'Deploy to server?'
                milestone(1)
                withCredentials ([usernamePassword(credentialsId: registryCredentials, usernameVariable: "USERNAME", passwordVariable: "USERPASS")]) {
                    sshPublisher(
                        continueOnError: false, failOnError: true,
                        publishers: [
                            sshPublisherDesc(
                                configName: "sandbox",
                                verbose: true,
                                transfers: [
                                    sshTransfer(execCommand: "docker login -u $USERNAME -p $USERPASS")
                                ]
                            )
                        ]
                    )
                }
                sshPublisher(
                    continueOnError: true, failOnError: false,
                    publishers: [
                        sshPublisherDesc(
                            configName: "sandbox",
                            verbose: true,
                            transfers: [
                                sshTransfer(execCommand: "docker stop train-schedule"),
                                sshTransfer(execCommand: "docker rm train-schedule"),
                            ]
                        )
                    ]
                )
                sshPublisher(
                    continueOnError: false, failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: "sandbox",
                            verbose: true,
                            transfers: [
                                sshTransfer(execCommand: "docker run --restart always --name train-schedule -p 3000:3000 -d $imagename:${env.BUILD_NUMBER}")
                            ]
                        )
                    ]
                )
            }
        }
    }
}
