pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: python
    image: python:3.12-alpine
    command:
    - cat
    tty: true
  - name: sonar-scanner
    image: sonarsource/sonar-scanner-cli
    command:
    - cat
    tty: true
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
    securityContext:
      runAsUser: 0
    volumeMounts:
    - name: kubeconfig-secret
      mountPath: /.kube/config
      subPath: kubeconfig
  - name: dind
    image: docker:dind
    args: ["--registry-mirror=https://mirror.gcr.io", "--storage-driver=overlay2"]
    securityContext:
      privileged: true  # Needed to run Docker daemon
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""  # Disable TLS for simplicity
    volumeMounts:
    - name: docker-config
      mountPath: /etc/docker/daemon.json
      subPath: daemon.json  # Mount the file directly here
  volumes:
  - name: docker-config
    configMap:
      name: docker-daemon-config
  - name: kubeconfig-secret
    secret:
      secretName: kubeconfig-secret
'''
        }
    }
    stages {
        stage('Build - Tag - Push') {
            steps {
                container('dind') {
                    script{
                        dir('bookmyshow-app'){
                            sh 'docker --version'
                            sh 'sleep 20'
                            sh 'docker build -t nexus-service-for-docker-hosted-registry.bookmyshow-ns.svc.cluster.local:8085/my-repository/bookmyshowapp:v1 .'
                            sh 'docker image ls'
                        }
                    }
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                container('sonar-scanner') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=bookmyshow \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://my-sonarqube-sonarqube.school-ns.svc.cluster.local:9000 \
                          -Dsonar.token=sqp_119b2cededa281d7dd5ef7f6ed0f9aab51a8da92
                    '''
                }
            }
    }
}
