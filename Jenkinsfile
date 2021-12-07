pipeline {
    agent any
    stages {
        stage('Git clone') {
            steps {
                echo 'Clonning project from the GIT'
                deleteDir()
                git 'https://github.com/smart-cn/cicd-pipeline-train-schedule-jenkins.git'
            }
        }
        stage('Artifact build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
    }
}
