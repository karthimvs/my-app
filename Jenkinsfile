pipeline {
    agent {
        label 'docker-agent1'
    }

    stages {
        stage ('Git Pull') {
            steps {
                git 'https://github.com/javaparser/javaparser-maven-sample.git'
            }
        }

    }
}
