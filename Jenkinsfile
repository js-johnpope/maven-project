pipeline {
    agent any
    stages {
        stage('Build') {

            steps {
                withMaven(maven : 'nonprod-maven') {
                    sh 'mvn -B -DskipTests clean package'
                }
                // buildImage(name: 'nonprod-docker') {
                //     sh "docker build . -t tomcatwebapp:${env.BUILD_ID}"
                // }
                // docker.build("tomcatwebapp:${env.BUILD_ID}")
            }
        }
    }
}

node {
    def customImage = docker.build("tomcatwebapp:${env.BUILD_ID}")
    // customImage.push()
}