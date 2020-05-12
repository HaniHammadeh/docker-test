pipeline {
    agent any
    stages {
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("hanihammadeh/my_apache")
                    app.inside {
                        sh 'echo $(curl localhost:80)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_login') {
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
                    script {
                        sh "docker pull willbla/train-schedule:${env.BUILD_NUMBER}"
                        try {
                            sh "docker stop my_apache"
                            sh "docker rm my_apache"
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "docker run --restart always --name my_apache -p 3000:80 -d hanihammadeh/my_apache:${env.BUILD_NUMBER}"
                    }
                }
            }
        }
