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
                git branch: 'k8sdeploy',
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
        stage('Deploy new Docker image to staging server'){
            steps([$class: 'BapSshPromotionPublisherPlugin']) {
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
        stage('Deploy to production (k8s)'){
            steps([$class: 'BapSshPromotionPublisherPlugin']) {
                input 'Deploy to production?'
                milestone(2)
                withCredentials ([usernamePassword(credentialsId: registryCredentials, usernameVariable: "USERNAME", passwordVariable: "USERPASS")]) {
                    sshPublisher(
                        continueOnError: false, failOnError: true,
                        publishers: [
                            sshPublisherDesc(
                                configName: "k8s-prod",
                                verbose: true,
                                transfers: [
                                    sshTransfer(execCommand: "kubectl create secret docker-registry train-docker-credentials --docker-username=$USERNAME --docker-password=$USERPASS || (kubectl delete secret train-docker-credentials && kubectl create secret docker-registry train-docker-credentials --docker-username=$USERNAME --docker-password=$USERPASS  || exit 1)"),
                                    sshTransfer(sourceFiles: 'trains-k8s.yaml', remoteDirectory: '$BUILD_TAG'),
                                    sshTransfer(execCommand: "sed -i 's/{{IMAGE_TAG}}/$BUILD_NUMBER/g' $BUILD_TAG/trains-k8s.yaml"),
                                    sshTransfer(execCommand: "kubectl apply -f $BUILD_TAG/trains-k8s.yaml"),
                                    sshTransfer(execCommand: "rm $BUILD_TAG/trains-k8s.yaml && rm -d $BUILD_TAG")
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}

