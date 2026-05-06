pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
    labels:
        app: kaniko
spec:
    containers:
    - name: kaniko
      image: gcr.io/kaniko-project/executor:debug
      workingDir: /home/jenkins/agent
      command:
        - sleep
      args:
        - 9999999
      volumeMounts:
        - name: docker-config
          mountPath: /kaniko/.docker/

    # kubectl 컨테이너 추가
    # kubectl 컨테이너는 Docker 이미지 빌드/푸시만 담당하고,
    # Kubernets 배포 재시작 명령은 kubectl이 필요하므로 별도 컨테이너를 추가
    - name: kubectl
      image: alpine/k8s:1.32.0
      workingDir: /home/jenkins/agent
        command:
          - sleep
        args:
          - 9999999

    volumes:
      - name: docker-config
        secret:
          secretName: dockerhub-secret
          items:
            - key: .dockerconfigjson
              path: config.json
"""
        }
    }

    stages {
        stage('Build & Push Backend') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor \
                      --context=${WORKSPACE}/backend \
                      --dockerfile=${WORKSPACE}/backend/Dockerfile \
                      --destination=whiteyan/beatbuddy-backend:dev
                    '''
                }
            }
        }

        stage('Build & Push Frontend') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor \
                      --context=${WORKSPACE}/frontend \
                      --dockerfile=${WORKSPACE}/frontend/Dockerfile \
                      --destination=whiteyan/beatbuddy-frontend:dev
                    '''
                }
            }
        }

        // Kubernetes 자동 재배포 단계
        // backend/frontend 이미지를 Docker Hub에 push한 뒤,
        // Kubernetes Deployment를 재시작해서 새 이미지를 다시 pull 하게 만듬
        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                    # backend Deployment 재시작
                    kubectl rollout restart deployment/beatbuddy-backend -n default

                    # frontend Deployment 재시작
                    kubectl rollout restart deployment/beatbuddy-frontend -n default

                    # backend 재배포 완료 여부 확인
                    kubectl rollout status deployment/beatbuddy-backend -n default

                    # frontend 재배포 완료 여부 확인
                    kubectl rollout status deployment/beatbuddy-frontend -n default
                    '''
                }
            }
        }
    }
}