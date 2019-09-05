podTemplate(
    label: 'slave-pod',
    inheritFrom: 'default',
    containers: [
        containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'kubectl', image: 'bitnami/kubectl:latest', ttyEnabled: true, command: 'cat'),
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
            serviceName = "tomcatwebapp"
            gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
            DOCKER_IMAGE_REPO = "649636635951.dkr.ecr.eu-west-1.amazonaws.com/${serviceName}"
            container ('docker') {
                withDockerRegistry([credentialsId: 'ecr:eu-west-1:5f67fb6c-f993-46ef-af99-9c1a99833f46', url: "https://${DOCKER_IMAGE_REPO}"]) {
                    sh """
                      docker build -t ${serviceName}:${gitCommit} .
                      docker tag ${serviceName}:${gitCommit} ${DOCKER_IMAGE_REPO}:${gitCommit}
                      docker tag ${serviceName}:${gitCommit} ${DOCKER_IMAGE_REPO}:latest
                      docker push ${DOCKER_IMAGE_REPO}:${gitCommit}
                      docker push ${DOCKER_IMAGE_REPO}:latest
                      """
                }
            }
        }

        stage ('Deploy to Kubernetes cluster') {
            container ('kubectl') {
                sh """
                  kubectl apply -f ./deployment.yaml
                  """
            }
        }
    }
}