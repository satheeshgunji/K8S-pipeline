   pipeline {
    agent any
    tools {
		maven 'Maven home'
	}
	environment {
      DOCKER_TAG = getVersion()
    }
    stages {
        stage('SCM Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', 
				url: 'https://github.com/satheeshgunji/K8S-pipeline.git'
            }
        }
        stage('Maven-Steps') {
            steps {
				sh 'mvn clean package '
            }
        }
        stage('Docker-Build') {
            steps {
				sh 'docker build . -t satheeshgunji/nodeapp:${DOCKER_TAG}'
            }
        }
        stage('Docker-Push') {
            steps {
				withCredentials([string(credentialsId: 'dockerpass', variable: 'dockerhubpassword')]) {
					sh 'docker login -u satheeshgunji -p ${dockerhubpassword}'
				}
			sh 'docker push satheeshgunji/nodeapp:${DOCKER_TAG}'
            }
        }
        stage('Deploy to k8s'){
            steps{
              sh "chmod +x changeTag.sh"
              sh "./changeTag.sh ${DOCKER_TAG}"
			  sshagent(['K8S-Mgmt-Node']) {
					sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ec2-user@54.208.14.246:/home/ec2-user"
                    sh "ssh ec2-user@54.208.14.246 kubectl apply -f ."
				}
            }
        }
    }
}
def getVersion(){
    def commitHash = sh returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
