podTemplate(containers: [
    containerTemplate(image: 'docker', name: 'docker', command: 'cat', ttyEnabled: true),
    containerTemplate(image: 'maven', name: 'maven', command: 'cat', ttyEnabled: true)
    ]) {

        // pipeline {
        //     agent any
    node(POD_LABEL) {
        stages {
            stage('Build') {
                steps {
                    container('maven') {
                        sh 'mvn -B -DskipTests clean package'
                    }
                    container('docker') {
                        sh "docker build . -t tomcatwebapp:${env.BUILD_ID}"
                    }
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

