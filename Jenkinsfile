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
    serviceAccountName: jenkins
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

    # kubectl 컨테이너
    # Kubernetes Deployment 재시작/상태확인을 위해 별도 kubectl 컨테이너를 사용한다.
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
                      --build-arg VITE_API_BASE_URL=http://localhost:31080 \
                      --destination=whiteyan/beatbuddy-frontend:dev
                    '''
                }
            }
        }

        // Kubernetes 자동 재배포 단계
        // 이미지 push 후 Deployment를 재시작해서 새 이미지를 다시 pull 하게 한다.
        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                    kubectl rollout restart deployment/beatbuddy-backend -n default
                    kubectl rollout restart deployment/beatbuddy-frontend -n default

                    kubectl rollout status deployment/beatbuddy-backend -n default
                    kubectl rollout status deployment/beatbuddy-frontend -n default
                    '''
                }
            }
        }
    }
}