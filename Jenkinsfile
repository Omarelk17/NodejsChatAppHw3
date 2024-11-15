pipeline {
    agent none
    stages {
        stage('CLONE GIT REPOSITORY') {
            agent {
                label 'nodejschatapp'
            }
            steps {
                checkout scm
            }
        }  
 
        stage('SCA-SAST-SNYK-TEST') {
            agent any
            steps {
                script {
                    snykSecurity(
                        snykInstallation: 'Snyk',
                        snykTokenId: 'snyk_api_token',
                        severity: 'critical'
                    )
                }
            }
        }
 
        stage('SonarQube Analysis') {
            agent {
                label 'nodejschatapp'
            }
            steps {
                script {
                    def scannerHome = tool 'SonarQube'
                    withSonarQubeEnv('SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=nodejschatapp \
                            -Dsonar.sources=."
                    }
                }
            }
        }
 
        stage('BUILD-AND-TAG') {
            agent {
                label 'nodejschatapp'
            }
            steps {
                script {
                    def app = docker.build("omarelk18/nodejschatapp")
                    app.tag("latest")
                }
            }
        }
 
        stage('POST-TO-DOCKERHUB') {    
            agent {
                label 'nodejschatapp'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_credentials') {
                        def app = docker.image("omarelk18/nodejschatapp")
                        app.push("latest")
                    }
                }
            }
        }
 
        stage('DEPLOYMENT') {    
            agent {
                label 'nodejschatapp'
            }
            steps {
                sh "docker-compose down"
                sh "docker-compose up -d"   
            }
        }
    }
}