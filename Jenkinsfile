pipeline {
    agent any
    environment {
        BUILD_NUMBER = "2.01"  // 빌드 번호
        IMAGE_NAME = "192.168.1.183:443/cicd-web/front-cicd"  // Harbor이미지 이름
        HARBOR_CREDENTIALS = credentials('harbor') // jenkins에 등록한 Harbor Credentials ID
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    credentialsId: 'github_access_token',  // 미리 설정한 GitHub 자격증명 ID
                    url: 'https://github.com/si-naeng/cicd-web.git'  // 내 Git URL
            }
        }
        stage('Login to Harbor') {
            steps {
                script {
                    // Harbor  Hub 로그인
                    sh "docker login -u ${HARBOR_CREDENTIALS_USR} -p ${HARBOR_CREDENTIALS_PSW} 192.168.1.183:443"
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                dir('web-2.0/frontend'){
                    echo "Start to Build the Image"
                    // Docker 이미지 빌드
                    sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
                    echo "Build Success"
                }
            }
        }
        stage('Push Image to HUB') {
            steps {
                script {
                    echo "Push to Harbor"
                    // Docker 이미지를 Docker Hub에 푸시
                    sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                    echo "Push Success"
                }
            }
        }
        stage('K8S Manifest Update') {
            steps {
                // GitHub에서 Kubernetes manifest 레포지토리 체크아웃
                git credentialsId: 'github_access_token',
                    url: 'https://github.com/si-naeng/cicd-web.git',
                    branch: 'master'
                sh 'git config user.email "jenkins@yourdomain.com"'
                sh 'git config user.name "Jenkins CI"'
                // deployment.yaml 파일의 버전 정보를 현재 빌드 번호로 업데이트
                // Git 변경사항 추가
                dir('manifests') {
                    sh """
                        sed -i 's|image: 192.168.1.183:443/cicd-web/front-cicd:.*|image: 192.168.1.183:443/cicd-web/front-cicd:${BUILD_NUMBER}|g' deploy.yaml
                        git add deploy.yaml
                        git commit -m '[UPDATE] my-app ${BUILD_NUMBER} image versioning'
                    """
                }

                // SSH로 GitHub에 푸시
                sshagent(credentials: ['k8s-manifest-credential']) {
                    sh "git remote set-url origin git@github.com:si-naeng/cicd-web.git"
                    sh "git push -u origin master"
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}
