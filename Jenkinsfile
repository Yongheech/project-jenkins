pipeline {
    agent any

    environment {
        DOCKER_IMAGE_OWNER = 'chkion1234'
        DOCKER_BUILD_TAG = "20241108.${env.BUILD_NUMBER}"
        DOCKER_IMAGE_TAG = 'latest'
        DOCKER_TOKEN = credentials('dockerhub') // Docker Hub 자격 증명
        GIT_CREDENTIALS = credentials('github_access_token')
        REPO_URL = 'Yongheech/project-jenkins.git'
        ARGOCD_REPO_URL = 'Yongheech/project-argocd.git'
        COMMIT_MESSAGE = 'Update README.md via Jenkins Pipeline'
    }

    stages {
        stage('Clone from SCM') {
            steps {
                dir('project-jenkins ') {
                    git branch: 'master', url: "https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${REPO_URL}"
                }
                dir('project-argocd') {
                    git branch: 'master', url: "https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${ARGOCD_REPO_URL}"
                }
            }
        }

        stage('Docker Image Building') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG} ./frontend"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG} ./frontend"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG} ./admin-service"
                    sh "docker build -t ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG} ./visitor-service"
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    sh "echo ${DOCKER_TOKEN_PSW} | docker login -u ${DOCKER_TOKEN_USR} --password-stdin"
                }
            }
        }

        stage('Docker Image Pushing') {
            steps {
                script {
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG}"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG}"
                    sh "docker push ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG}"
                }
            }
        }

        stage('Update ArgoCD Deployment YAML with Image Tags') {
            steps {
                dir('project-argocd') {
                    sh """
                    sed -i "s|image: {{.Values.image.admin.repository}}|image: ${DOCKER_IMAGE_OWNER}/prj-admin:${DOCKER_BUILD_TAG}|g" deploy-argocd/templates/deployment.yaml
                    sed -i "s|image: {{.Values.image.visitor.repository}}|image: ${DOCKER_IMAGE_OWNER}/prj-visitor:${DOCKER_BUILD_TAG}|g" deploy-argocd/templates/deployment.yaml
                    sed -i "s|image: {{.Values.image.frontend.repository}}|image: ${DOCKER_IMAGE_OWNER}/prj-frontend:${DOCKER_BUILD_TAG}|g" deploy-argocd/templates/deployment.yaml
                    """

                }
            }
        }
        
        stage('Commit Changes') {
            steps {
                dir('project-argocd') {
                    script {
                        def changes = sh(script: "git status --porcelain", returnStdout: true).trim()
                        if (changes) {
                            sh '''
                            git config user.name "Yongheech"
                            git config user.email "alpacatnt@jenkins.com"
                            git add deploy-argocd/templates/deployment.yaml
                            git commit -m "Update image tags to ${DOCKER_BUILD_TAG}"
                            '''
                        } else {
                            echo "No changes to commit"
                        }
                    }
                }
            }
        }

        stage('Push Changes') {
            steps {
                dir('finalprojectargocd') {
                    script {
                        sh "git push https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/${ARGOCD_REPO_URL} main"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
