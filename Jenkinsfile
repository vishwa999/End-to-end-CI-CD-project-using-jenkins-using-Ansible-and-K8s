
def getDockerTag(){
  def tag = sh script: 'git rev-parse HEAD', returnStdout: true
  return tag
}
pipeline{
        
           agent any
           environment{
	           Docker_tag = getDockerTag()
        }
       
        stages{
                 stage('Quality Gate Statuc Check'){
                   agent {
                        docker {
                                image 'maven'
                                args '-v $HOME/.m2:/root/.m2:z -u root'
                                reuseNode true
                              }
                          }
                   steps{
                           script{
                                   withSonarQubeEnv('sonarserver') 
                                    { 
                                     sh "mvn sonar:sonar"
                                   }
                           timeout(time: 1, unit: 'HOURS') {
                           def qg = waitForQualityGate()
                          if (qg.status != 'OK') {
                                 error "Pipeline aborted due to quality gate failure: ${qg.status}"
                           }
                        }
		                    sh "mvn clean install"
                    }
                 }  
                }
               stage("Build Docker image"){
                steps{
                  script{
                       sh 'docker build . -t ski00026/end-to-end-k8s:$Docker_tag'
                       withCredentials([string(credentialsId: 'Docker_hub', variable: 'DockerHubPassword')]) {
                                       sh 'docker login -u ski00026 -p $DockerHubPassword'
                                      sh 'docker push ski00026/end-to-end-k8s:Docker_tag'
                                  }
                       
                   }
                 }
               }
             stage('ansible playbook'){
			          steps{
			 	            script{
				               sh '''final_tag=$(echo $Docker_tag | tr -d ' ')
				               echo ${final_tag}test
				               sed -i "s/docker_tag/$final_tag/g"  deployment.yaml
                       '''
				                 ansiblePlaybook become: true, installation: 'ansible', inventory: 'hosts', playbook: 'ansible.yaml'
				              }
			              }
		            }
	   }
}