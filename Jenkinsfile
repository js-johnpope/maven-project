pipeline {
    agent {
        kubernetes {
            containerTemplate {
                name 'docker'
                image 'docker:1.11'
                ttyEnabled true
                command 'cat'
            }
            containerTemplate {
                name 'maven'
                image maven:latest
                command 'cat'
                ttyEnabled true
            }
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

