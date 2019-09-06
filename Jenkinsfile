podTemplate(
    label: 'slave-pod',
    inheritFrom: 'default',
    containers: [
        containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
        // containerTemplate(name: 'kubectl', image: 'bitnami/kubectl:1.15.3-debian-9-r15', ttyEnabled: true, command: 'cat'),
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
                withKubeConfig(credentialsId: 'jenkins-k8s-deployer-credentials',
                    contextName: 'AssumeRoleSession@GMCP-Integration-Dev.eu-west-1.eksctl.io',
                    clusterName: 'AssumeRoleSession@GMCP-Integration-Dev.eu-west-1.eksctl.io',
                    serverUrl: 'https://FE0F14D2721D63D2ABB698A4062DB933.sk1.eu-west-1.eks.amazonaws.com',
                    caCertificate: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRFNU1EZ3lPREEzTURjeE9Gb1hEVEk1TURneU5UQTNNRGN4T0Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTXY3ClhDZGRqWCtTODlDTHBMZkYzdG1oMUhyUFMzdnZMOW1JbVVhbmlZd2UvaThRWnJqazF2Z3dnNlNTK1ZaQXJBNjAKZ0ttNzJNazNHa256YnVrWkhienlLVDZrWDJxLy9aVU5SWXJueUQ1TVVzeHRTeVkrZDV6RXZjVTFSM0ErOVhjMwpBTjI5b1JYWFpkRDhqWmpVVTlRZWVjdm1lcDQ5eXRCSlBTWmc4Q1k3eWRIWlRVSnBGSjV5WVVuTk1MMzFBMHVXCkcvZFkydXZEc0NHcWZ5aVQ0UWxxWVJQSHZ0WVh1MWpqbGJPY0xxVkVjcjJvMGw4My8wZk5pdGl2bFJOdWkzV1EKS1duOW5lMWFhMUdmR1VYRC9jZDhPa0RNYjQrQjNqaFQyUHJXdlRKUytmT0tCbkhmRFN0Nlh1VzVMMFVLbjZPWAo2TGFNSks3SVhuejdlZk1jQ2djQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFNcFBZTzlXT2pBVFJUNXVMc3pqSm5PczdDSW0KS0p5Tm1GUFZOZDFMdEJYZzlpckZEVU9pODg0L3JHQ0hVMThoTE5YVlZoaDNYdC9oeGZaSUt4RmkxWE80L0VvLwpOc2Rxemg3UWtIYnZYTnVLNzV3ZExlSXk4Y0NTVWl6WWUyOHlwOFRQd2w2VFJEeW1zUHFrcGVHeWJsS0dOR0RKCmVOTm5uckQxYXpYbVFSOVN3ZG05eEJjZHNFbFQwM2xxV0NIaHlPZmY5RzNyZU5MS0Vjbk9QV3JvNlduSFQzSTUKZW1CRGgwNUVaQVhrYUI0dFNyWWN4YThUM3puS1BwcWRudDExQ0F5QzNGSmpaYU5GMEFOR3Y0T25GVU9wWDlDOApwZlNiaEhiM1hRdHk0WVdxakJRUXh4RmpoQ040T0syQ084TFdHTWlzTHRhTXhrd1FhQkYyQVpudkRuST0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=') {
                    sh """
                      kubectl get pods -n jenkins
                      kubectl apply -f ./deployment.yaml -n apps
                      """
                }
            }
        }
    }
}