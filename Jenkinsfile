
pipeline {
 agent any
  stages {
    stage('SCM') {
      steps {
         git 'https://github.com/Debadutta-Pradhan/Voting_App.git'
       }
    }
   stage("Build") {
    tools{
        jdk 'Jdk_1.8'
        maven 'apache-maven-3.6.3'
        }
     steps {
    bat '''
           cd worker
           mvn clean install
	   java -version
           mvn -version
           mvn clean package
                   
       '''
   }
   }
   stage('SonarQube') {
                steps {
                   bat '''
                     cd worker
                     mvn sonar:sonar \
			  -Dsonar.projectKey=Voting_Map \
			  -Dsonar.host.url=http://localhost:9000 \
			  -Dsonar.login=a9393aaba784cfc4931e732eb03f9911f870aea6
                   '''
   }
   }
   stage('Approval') {
            // no agent, so executors are not used up when waiting for approvals
            agent none
            steps {
                script {
                    def deploymentDelay = input id: 'Deploy', message: 'Deploy to production?', submitter: 'rkivisto,admin', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
                    sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
                }
            }
        }
	  stage('Build image') {
                   steps {
                    echo 'Starting to build docker image DB'
                    script {
                        def DB = docker.build("my-image:${env.BUILD_ID}","-f ${env.WORKSPACE}/vote/Dockerfile .")
                        def nodejs = docker.build("my-image:${env.BUILD_ID}","-f ${env.WORKSPACE}/result/Dockerfile .") 
                        def php = docker.build("my-image:${env.BUILD_ID}","-f ${env.WORKSPACE}/worker/Dockerfile .") 
                    }
                }
            }
}
post {
        always {
            //archiveArtifacts artifacts: 'generatedFile.log', onlyIfSuccessful: true
          
            emailext attachLog: true,
                body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
                recipientProviders: [developers(), requestor()],
                subject: "Jenkins Build :- ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
            
        }
    }
}



