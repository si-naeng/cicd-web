pipeline {
    agent any
    environment {
        BUILD_NUMBER = "2.02"  // 빌드 번호
        IMAGE_NAME = "192.168.1.183:443/cicd-web/front-cicd"  // Harbor 이미지 이름
        HARBOR_CREDENTIALS = credentials('harbor')  // Harbor Credentials ID
        GITHUB_CREDENTIALS = credentials('github_access_token')  // GitHub Credentials ID
    }
    stages {
        stage('Clone repository') {
            steps {
                git branch: 'master',  // 사용하려는 브랜치
                    credentialsId: 'github_access_token',  // GitHub 자격증명 ID
                    url: 'https://github.com/si-naeng/cicd-web.git'  // 내 Git URL
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -f web-2.0/frontend/Dockerfile -t ${IMAGE_NAME}:${BUILD_NUMBER} web-2.0/frontend"
                }
            }
        }

        stage('Push Docker Image to Harbor') {
            steps {
                script {
                    sh "echo ${HARBOR_CREDENTIALS_PSW} | docker login -u ${HARBOR_CREDENTIALS_USR} --password-stdin 192.168.1.183:443"
                    sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                script {
                    sh "git config user.email 'koo2813@naver.com'"  // GitHub 이메일
                    sh "git config user.name 'si-naeng'"  // GitHub 사용자 이름

                    sh """
                        sed -i 's|image: .*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|g' manifests/cicd-deploy.yaml
                    """
                    sh "git add manifests/cicd-deploy.yaml"
                    sh "git commit -m '[UPDATE] Updated to image version ${BUILD_NUMBER}'"

                    // Personal Access Token을 사용하여 안전하게 Push
                    sh """
                        git push https://${env.GITHUB_CREDENTIALS_USR}:${env.GITHUB_CREDENTIALS_PSW}@github.com/si-naeng/cicd-web.git master
                    """
                }
            }
        }
    }
    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline execution failed!"
        }
    }
}

