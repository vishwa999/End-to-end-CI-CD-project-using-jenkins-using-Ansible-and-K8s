
pipeline{
        agent {
                docker {
                  image 'maven:3-alpine'
                  args '-v $HOME/.m2:/root/.m2:z -u root'
                  reuseNode true
                }
             }
       
        stages{
              stage('Quality Gate Statuc Check'){

                  steps{
                      script{
                      withSonarQubeEnv('sonarserver')  { 
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
              
              
      }
    }
