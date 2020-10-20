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
            steps {
                script {
                    app = docker.build("mcweenjr/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image!') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_id') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            steps {
                input 'Deploy to Production?'
                milestone(1)
                    script {
                        try {
                            sh 'docker stop train-schedule'
                            sh 'docker rm train-schedule'
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh 'docker run --restart always --name train-schedule -p 8081:8080 -d train-schedule:${env.BUILD_NUMBER}'
                    }
                }
            }
        }
    }
