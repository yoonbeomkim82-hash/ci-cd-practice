pipeline {
    agent any // Jenkins Agent를 사용합니다.

    // 🚨 환경 변수 설정: 반드시 본인의 ID로 변경하세요!
    environment {
        DOCKER_ID = 'randy857' 
        IMAGE_NAME = 'my-ci-cd-app'
        K8S_DEPLOYMENT_FILE = 'k8s-deployment.yaml'
    }

    stages {
        stage('Source Checkout') {
            steps {
                // Git 플러그인이 자동으로 코드를 체크아웃합니다.
                echo "Source code checked out."
            }
        }
        
        stage('Build & Test') {
            steps {
                echo 'CI: Running npm install...'
                sh 'npm install'
                echo 'CI: Running tests (placeholder)...'
                sh 'npm test' // package.json의 test 스크립트 실행
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def appImageTag = "${DOCKER_ID}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
                    
                    // Docker 이미지 빌드
                    sh "docker build -t ${appImageTag} ."
                    
                    // Docker Hub 로그인 (Jenkins Credential 'docker-hub-creds' 필요)
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                        
                        // Docker Hub에 이미지 푸시
                        sh "docker push ${appImageTag}"
                        sh "docker tag ${appImageTag} ${DOCKER_ID}/${IMAGE_NAME}:latest" // latest 태그도 업데이트
                        sh "docker push ${DOCKER_ID}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }
        
        stage('Deploy to K8s') {
            steps {
                echo 'CD: Deploying to Kubernetes...'
                
                // K8s YAML 파일의 이미지 태그를 현재 빌드 번호로 변경
                // sed 명령은 현재 빌드 번호로 이미지를 업데이트하여 배포가 Rolling Update되게 합니다.
                sh "sed -i '' 's|${DOCKER_ID}/${IMAGE_NAME}:.*|${DOCKER_ID}/${IMAGE_NAME}:${env.BUILD_NUMBER}|g' ${K8S_DEPLOYMENT_FILE}"

                // K8s에 배포 적용 (kubectl 명령 실행)
                sh "kubectl apply -f ${K8S_DEPLOYMENT_FILE}"
            }
        }
    }
}