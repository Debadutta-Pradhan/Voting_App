
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
                             -Dsonar.projectKey=186ff1c937cbd8e4334413f9512ac2860973d79c \
                             -Dsonar.host.url=http://localhost:9000 \
                             -Dsonar.login=7c8f06e9919bef784b20055108989b17dfdc85c2
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
   stage('Deploy') {
                  steps{
				    echo "Deploying"
				    deploy adapters: [tomcat7(credentialsId: 'd8511587-aae6-4afd-ad92-782639ac4b60', path: '', url: 'http://localhost:8090')], contextPath: 'happytrip', war: '**/*.war'
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



