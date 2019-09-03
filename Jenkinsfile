pipeline {
    agent any
    stages {
        stage('Build') {

            steps {
                withMaven(maven : 'nonprod-maven') {
                    sh 'mvn -B -DskipTests clean package'
                }
                // withDocker(docker: 'nonprod-docker') {
                sh "docker build . -t tomcatwebapp:${env.BUILD_ID}"
                }
            }
        }
    }
}