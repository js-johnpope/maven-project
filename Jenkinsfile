podTemplate(
    label: 'slave-pod',
    inheritFrom: 'default',
    containers: [
        containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'docker', image: 'docker:18.02', ttyEnabled: true, command: 'cat')
    ],
    volumes: [
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
        hostPathVolume(hostPath: '/root/.m2', mountPath: '/root/.m2')
    ]
) {
    node('slave-pod') {
        def commitId
        stage ('Extract') {
            checkout scm
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }

        stage ('Build') {
            container ('maven') {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage ('Docker build and push') {
            container ('docker') {
                // def repository = "sybrenbolandit/java-spring-api"

                // withCredentials([usernamePassword(credentialsId: 'dockerhub',
                //         usernameVariable: 'registryUser', passwordVariable: 'registryPassword')]) {

                    // sh "docker login -u=$registryUser -p=$registryPassword"
                    sh "docker build -t tomcatwebapp:${commitId} ."
                    // sh "docker push ${repository}:${commitId}"
                }
            }
        }
    }
}