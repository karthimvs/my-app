pipeline {
    agent {
        label 'docker-agent1'
    }
    
    tools {
        maven3 'Maven 3.6.1'
    }

    stages {
        stage ('Git Pull') {
            steps {
                git 'https://github.com/javaparser/javaparser-maven-sample.git'
            }
        }
        
        stage ('Compile Maven') {
            steps {
                sh "mvn clean package"
                sh 'mv target/myweb*.war target/app1.war'
            }
        }

    }
}
