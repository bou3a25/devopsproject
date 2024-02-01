pipeline {
    agent { label 'agent1'}
    tools {git 'Default'}

    environment {
        dockerImage = "my_image" 
        gitlab_repo = "root/project1"
        sonarProjectKey = 'project1'

        gitlab_lien = "http://192.168.111.15/${gitlab_repo}.git"
        gitlab_registry = "192.168.111.15:5005"
        gitlab_registry_build = "${gitlab_registry}/${gitlab_repo}/${dockerImage}:${BUILD_NUMBER}"
        gitlab_registry_push = "http://${gitlab_registry}/${gitlab_repo}"
    }
    stages {

        stage('Checkout from GitLab') {
            steps {
                git credentialsId: 'api', url: "${gitlab_lien}"
            }
        }

     

        stage('Run SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('sonar') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=${sonarProjectKey}"
                    }
                }
            }
        }
        
        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build("${gitlab_registry_build}")
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry("${gitlab_registry_push}", 'api') {
                        dockerImage.push()
                    }
                }
            }
        }

    }
}
