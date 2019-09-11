#!groovy

def handleCheckout = {
    sh "echo 'Checking out given branch...'"
    checkout scm
}

podTemplate(
    label: 'slave-pod',
    inheritFrom: 'default',
    containers: [
        containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'kubectl', image: 'amaceog/kubectl', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'docker', image: 'docker:18.02', ttyEnabled: true, command: 'cat')
    ],
    volumes: [
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),
        hostPathVolume(hostPath: '/root/.m2', mountPath: '/root/.m2'),
        hostPathVolume(hostPath: '/root/.kube', mountPath: '/root/.kube')
    ]
) {
    node('slave-pod') {
        def commitId
        stage ('Extract') {
            handleCheckout()
            commitId = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }

        stage ('Test') {
            container ('maven') {
                sh 'mvn test'
            }
        }

        stage ('Build') {
            container ('maven') {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage ('Docker build and push to ECR') {
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
    }

    input 'Do you approve deployment?'

    node ('slave-pod') {
        stage ('Deploy to Kubernetes cluster') {
            container ('kubectl') {
                withKubeConfig(credentialsId: 'jenkins-k8s-deployer-credentials',
                    serverUrl: 'https://FE0F14D2721D63D2ABB698A4062DB933.sk1.eu-west-1.eks.amazonaws.com') {
                    sh """
                      kubectl get pods -n jenkins
                      kubectl apply -f ./deployment.yaml -n apps
                      """
                }
            }
        }
    }
}