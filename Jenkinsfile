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
        
        stage (Compile Maven) {
            steps {
                def mvnHome = tool 'maven3', type: 'maven'
                sh "${mvnHome}/bin/mvn clean package"
                sh 'mv target/myweb*.war target/app1.war'
            }
        }

    }
}
