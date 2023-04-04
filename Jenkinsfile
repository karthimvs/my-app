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

        stage ("Compile Maven") {
            steps {
                def mvnHome = tool name: 'maven3', type: 'maven'
                sh "${mvnHome}/bin/mvn clean package"
                    sh 'mv target/myweb*.war target/app1.war'
            }
       }

        stage ("Send SonarQube Analysis") {
            steps {
                def scannerHome = tool 'sonarqube'
                withSonarQubeEnv('sonar') {
                    sh "${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=intern-java-project \
                    -Dsonar.login=5fbe28051908c3f10d88da4f6ec22c6a40ab8168"
            }
           }
        }

        stage ("Docker Build") {
            steps {
                sh "docker build -t interproject/inter-v1 ."
            }
        }

        stage ("Docker Deploy - Test Environment") {
            steps {
                sh "docker run -itd -p 9955:8080 -name Inter Project interproject/inter-v1"
            }
        }

        stage ("Get Approve") {
            steps {
                input "Approval for to deploy production Server"
            }
        }
    }
}
