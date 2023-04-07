pipeline {
    agent {
        label 'docker-agent1'
    }

    environment {
        AWS_ACCOUNT_ID = 'Enter your ID Here'
        SONAR_PROJECT_KEY = "Enter your sonarqube project Key Here"
        SONAR_PROJECT_NAME = "ENter your Project Name Here"
        BUILD_VERSION = "Enter your build Version"
        NEXUS_SERVER_TAG = "Enter your Nexus Server TAG Here"
        NEXUS_LOGIN_PASSWD = "Enter Nexus Login Password"
    }

    stages {
        stage ('SCM Checkout') {
            steps {
                git 'https://github.com/karthimvs/my-app.git'
            }
        }	
		        
        stage ('Compile Maven') {
            steps {
                script {
                    def mvnHome = tool name: 'maven3.6', type: 'maven'
                    sh "${mvnHome}/bin/mvn clean package"
                    sh 'mv target/myweb*.war target/app1.war'
                }
            }
        }
             
        stage('SonarQube Analysis') {
            steps {
                script {
                    def mvnHome = tool name: 'maven3.6', type: 'maven'
                    withSonarQubeEnv('sonar') {
                        sh "${mvnHome}/bin/mvn sonar:sonar \
			-Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                    	-Dsonar.login=${SONAR_PROJECT_NAME}"
                    }
                }
            }
        }

        stage ('Docker Build') {
            steps {
                sh "docker build -t interproject/inter-v1 ."
            }
        }
	    
        stage ('Remove Existing Container') {
            steps {
                sh 'docker rm -f InterProject'
            }
        }

        stage ('Docker Deploy - Test Environment') {
            steps {
                sh "docker run -itd -p 9955:8080 --name InterProject interproject/inter-v1:latest"
            }
        }

        stage ('Get Approve') {
            steps {
                input "Approval for to deploy production Server"
            }
        }

        stage("ECR Image Tagging") {
            steps {
                script {
                    sh '''
                lastbuild0=$(docker images | grep -B 1 -w  -E '(^|\\s)interproject/inter-v1($|\\s)' | awk '{print $1}' | awk 'NR==2')
                lastbuild1=$(docker images | grep -B 1 -w  -E '(^|\\s)interproject/inter-v1($|\\s)' | awk '{print $2}' | awk 'NR==2')
                ecrtagfinal=$(echo $lastbuild0:$lastbuild1)
                docker tag $ecrtagfinal ${AWS_ACCOUNT_ID}.dkr.ecr.ap-south-1.amazonaws.com/inter:latest
                   '''
                }
            }
        }

        stage("Docker Push ECR") {
            steps {
                sh "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.ap-south-1.amazonaws.com"
                sh "docker push ${AWS_ACCOUNT_ID}5.dkr.ecr.ap-south-1.amazonaws.com/inter:latest"
            }
        }

        stage ('Production Deployment in AWS') {
            agent ansible
            steps {
                sh "ansible-playbook -i karthi-inventory ecs-fargate-deployment.yaml"
            }
        }

        stage("Nexus Image Tagging") {
            steps {
                script {
                    sh '''
                ecrlastbuild0=$(docker images | grep -B 1 -w  -E '(^|\\s)${AWS_ACCOUNT_ID}.dkr.ecr.ap-south-1.amazonaws.com/inter-v1($|\\s)' | awk '{print $1}' | awk 'NR==2')
                ecrlastbuild1=$(docker images | grep -B 1 -w  -E '(^|\\s)${AWS_ACCOUNT_ID}.ecr.ap-south-1.amazonaws.com/inter-v1($|\\s)' | awk '{print $2}' | awk 'NR==2')
                nexustagfinal=$(echo $ecrlastbuild0:$ecrlastbuild1)
                nexusfinalpush=$(docker tag $nexustagfinal ${NEXUS_SERVER_TAG}:8080/inter-v1:${BUILD_VERSION})
                   '''
                }
            }
        }

        stage("Docker Push Nexus") {
            steps {
                script {
                    sh '''
                    docker login -u 'dockerpush' -p ${NEXUS_LOGIN_PASSWD} ${NEXUS_SERVER_TAG}:8080
                    docker push ${NEXUS_SERVER_TAG}:8080/cpts:${BUILD_VERSION}
                       '''
                }
            }
        }
    }

/*** Workspace clean up*/
 post {
        always {
            cleanWs()
        }
    }
 }
