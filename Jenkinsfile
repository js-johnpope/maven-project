pipeline {
    agent any
    stages {
        stage('Build') {

            steps {
                container('jnlp') {
                // withMaven(maven : 'nonprod-maven') {
                    sh 'mvn -B -DskipTests clean package'
                    sh "docker build . -t tomcatwebapp:${env.BUILD_ID}"
                }
                // withDocker(docker: 'nonprod-docker') {
                // sh "docker build . -t tomcatwebapp:${env.BUILD_ID}"
                // }
            }
        }
    }
}