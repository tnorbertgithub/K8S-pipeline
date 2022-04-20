pipeline {
    agent any
    tools {
        maven 'maven home'
           }
    environment {
      DOCKER_TAG = getVersion()
    }
    stages {
        stage('SCM-Checkout') { 
            steps {
				git branch: 'main', credentialsId: 'f191d72a-388e-4716-ba7d-fbef443b8cca', url: 'https://github.com/tnorbertgithub/K8S-pipeline.git'
            }
        }
        stage('Maven-package'){
            steps {
                sh "mvn clean compile package"
            }
        }
        stage('Docker-Build'){
            steps {
                sh "docker build . -t tnorbert/nodeapp:${DOCKER_TAG} "
            }
        }
        stage('Docker Push') { 
            steps {
				withCredentials([string(credentialsId: 'DockerHubCredential', variable: 'DockerHubPwd')]) {
				sh "docker login -u tnorbert -p ${DockerHubpwd}"
		        sh "docker push tnorbert/nodeapp:${DOCKER_TAG} "
				}
            }
        }
	stage('Ansible Deploy'){
            steps{
            ansiblePlaybook credentialsId: 'EC2-USER', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
    }
}
def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
