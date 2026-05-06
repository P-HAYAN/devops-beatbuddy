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
    }
}