pipeline {
    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    test-label: this-is-a-test-label
spec:
  containers:
  - name: maven
  - image: maven
    command: 
    - cat
    tty: true
  - name: docker
  - image: docker
    command: 
    - cat
    tty: true
"""
        }
    }

    stages {
        stage('Build') {
            steps {
                container('maven') {
                // withMaven(maven : 'nonprod-maven') {
                    sh 'mvn -B -DskipTests clean package'
                }
                container('docker') {
                    sh "docker build . -t tomcatwebapp:${env.BUILD_ID}"
                }
            }
        }
    }
}
                // withMaven(maven : 'nonprod-maven') {
                //     sh 'mvn -B -DskipTests clean package'
                // }
                // buildImage(name: 'nonprod-docker') {
                //     sh "docker build . -t tomcatwebapp:${env.BUILD_ID}"
                // }
                // docker.build("tomcatwebapp:${env.BUILD_ID}")

