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
        
        
        
        stage('SonarQube Analysis') {
            steps {
                script {
	                def mvnHome =  tool name: 'maven3', type: 'maven'
	                withSonarQubeEnv('sonar') { 
	                sh "${mvnHome}/bin/mvn sonar:sonar"
	                }
                }    
            }	    
        }
            

        stage ('Docker Build') {
            steps {
                sh "docker build -t interproject/inter-v1 ."
            }
        }

        stage ('Docker Deploy - Test Environment') {
            steps {
                sh "docker run -itd -p 9955:8080 -name Inter Project interproject/inter-v1"
            }
        }

        stage ('Get Approve') {
            steps {
                input "Approval for to deploy production Server"
            }
        } 

    }
}
