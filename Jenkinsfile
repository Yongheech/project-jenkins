pipeline {
    agent any

	environment{
		DOCKER_IMAGE_OWNER = 'chkion1234'
		DOCKER_IMAGE_TAG = 'latest'
		DOCKER_TOKEN = credentials('dockerhub')
	}

    stages {
        stage('clone from SCM') {
            steps {
                sh '''
                rm -rf project-jenkins
				git clone https://github.com/Yongheech/project-jenkins.git
				'''
            }
        }
		stage('Docker Image Building') {
            steps {
                sh '''
                cd project-jenkins
				docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG} ./frontend
				docker build -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG} ./admin-service
				docker build -t ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG} ./visitor-service
				'''
            }
        }
		stage('Docker Login') {
            steps {
				withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USR', passwordVariable: 'DOCKER_PWD')]) {
				sh "echo $DOCKER_PWD | docker login -u $DOCKER_USR --password-stdin"
				}
            }
        }
		stage('Docker Image pushing') {
            steps {
                sh '''
				docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG}
				docker push ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_IMAGE_TAG}
				docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_IMAGE_TAG}
				'''
            }
        }
        stage('Trigger ArgoCD') {
           steps {
               script {
                 sh '''
                 curl -k -X POST https://43.200.70.253:31714/api/webhook -H "Content-Type: application/json" -d '{"ref": "refs/heads/main"}'
                 '''
               }
           }
        }
		stage('Docker Logout') {
            steps {
                sh '''
				docker logout
				'''
            }
        }
    }
}