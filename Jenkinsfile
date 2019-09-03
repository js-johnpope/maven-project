pipeline {
    agent any
    stages {
        stage('Build') {

            steps {
                withMaven(maven : 'nonprod-maven') {
                    sh 'mvn -B -DskipTests clean package'
                }
            }
        }
    }
}