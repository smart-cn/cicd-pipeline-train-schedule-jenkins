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
                git branch: 'adddockerfile',
                    url: 'https://github.com/smart-cn/cicd-pipeline-train-schedule-jenkins.git'
            }
        }
        stage('Building Docker image') {
            steps{
                script {
                    dockerImage = docker.build imagename
                }
            }
        }
        stage('Deploy Docker Image') {
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
    }
}
