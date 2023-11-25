pipeline {
  agent any
  tools {
  maven 'MAVEN_HOME'
  }
    
	stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/olujlab/devops-a.git']]])
        }
      }
	  
	  stage ('Build')  {
	      steps {
            dir('webapp'){
            sh "pwd"
            sh "ls -lah"
            sh "mvn package"
          }
        }   
      }
   
     stage ('SonarQube Analysis') {
        steps {
              withSonarQubeEnv('sonar') {
				dir('webapp'){
                 sh 'mvn -U clean install sonar:sonar'
                }		
              }
            }
      }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog",
                    url: "http://3.209.132.235:8082/artifactory",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "project-a-libs-release-local",
                    snapshotRepo: "project-a-libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog", // credential ID from Jenkins global credentials
                    releaseRepo: "project-a-libs-release-local",
                    snapshotRepo: "project-a-libs-snapshot-local"
                )
            }
    }

    stage ('Deploy Artifacts') {
            steps {
                rtMavenRun (
                    tool: "MAVEN_HOME", // Tool name from Jenkins configuration
                    pom: 'webapp/pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
         }
    }

    stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "jfrog"
             )
        }
    }

    stage('Copy Dockerfile & Playbook to Staging Server') {
            
            steps {
                  sshagent(['ssh_agent']) {
                       sh "chmod 400  olujKP-NOVA.pem"
                       sh "ls -lah"
                        sh "scp -i olujKP-NOVA.pem -o StrictHostKeyChecking=no dockerfile ubuntu@34.199.39.176:/home/ubuntu"
                        sh "scp -i olujKP-NOVA.pem -o StrictHostKeyChecking=no dockerfile ubuntu@34.199.39.176:/home/ubuntu"
                        sh "scp -i olujKP-NOVA.pem -o StrictHostKeyChecking=no push-dockerhub.yaml ubuntu@34.199.39.176:/home/ubuntu"
                    }
                }
        } 

    stage('Build Container Image') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "ssh -i olujKP-NOVA.pem -o StrictHostKeyChecking=no ubuntu@34.199.39.176 -C \"ansible-playbook  -vvv -e build_number=${BUILD_NUMBER} push-dockerhub.yaml\""       
                    }
                }
        } 

    stage('Copy Deployment & Service Defination to K8s Master') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "scp -i olujKP-NOVA.pem -o StrictHostKeyChecking=no deploy_service.yaml ubuntu@18.214.65.9:/home/ubuntu"
                        }
                }
        } 

    stage('Waiting for Approvals') {
            
        steps{
		input('Test Completed ? Please provide  Approvals for Prod Release ?')
			 }
    }     
    stage('Deploy Artifacts to Production') {
            
            steps {
                  sshagent(['ssh_key']) {
                        //sh "ssh -i olujKP-NOVA.pem -o StrictHostKeyChecking=no ubuntu@18.214.65.9 -C \"kubectl set image deployment/ranty customcontainer=olujlab/july-set:${BUILD_NUMBER}\""
                        //sh "ssh -i olujKP-NOVA.pem -o StrictHostKeyChecking=no ubuntu@18.214.65.9 -C \"kubectl delete deployment ranty && kubectl delete service ranty\""
                        sh "ssh -i olujKP-NOVA.pem -o StrictHostKeyChecking=no ubuntu@18.214.65.9 -C \"kubectl apply -f deploy_service.yaml\""
                        //sh "ssh -i olujKP-NOVA.pem -o StrictHostKeyChecking=no ubuntu@18.214.65.9 -C \"kubectl apply -f service.yaml\""
                    }
                }  
        } 
   } 
}
