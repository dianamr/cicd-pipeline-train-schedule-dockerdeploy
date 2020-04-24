pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    sh " sudo docker build renatam/train-schedule"
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps{
                script {
                    docker.withRegistry ('https://registry.hub.docker.com', 'dockerhub_key'){
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)                       
                  sshagent (['prod-login']){
                    script {
                        sh "ssh -o StrictHostKeyChecking=no jenkins@10.166.0.3 \"docker pull renatam/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "ssh -o StrictHostKeyChecking=no jenkins@10.166.0.3 \"docker stop train-schedule\""
                            sh "ssh -o StrictHostKeyChecking=no jenkins@10.166.0.3 \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "ssh -o StrictHostKeyChecking=no jenkins@10.166.0.3 \"docker run --restart always --name train-schedule -p 8080:8080 -d renatam/train-schedule:${env.BUILD_NUMBER}\""
                        }                    
                 }
            }
        }
    }
}
