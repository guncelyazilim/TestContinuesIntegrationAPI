node  {
    def app
    def COLOR_MAP = ['SUCCESS': 'good', 'FAILURE': 'danger', 'UNSTABLE': 'danger', 'ABORTED': 'danger']
    try{
	stage('Restore packages'){
			bat "dotnet restore TestContinuesIntegrationAPI\\TestContinuesIntegrationAPI.csproj"
	
	}	
	stage('Clean'){
	
			bat "dotnet clean TestContinuesIntegrationAPI\\TestContinuesIntegrationAPI.csproj"
		 
    }
	stage('Build'){
		  bat "dotnet build TestContinuesIntegrationAPI\\TestContinuesIntegrationAPI.csproj --configuration Release"
		
    }
	stage('Publish'){
       bat "dotnet publish TestContinuesIntegrationAPI\\TestContinuesIntegrationAPI.csproj "
     
	}
    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */
            checkout scm
        }
    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */
            app = docker.build("serenkaraca/testcontinuesintegrationapi")
        }
    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('', 'DockerHubAccount') {
            app.push("latest")
      }
    }

    stage('Restart Application') {
            sh "docker-compose down"
            sh "docker-compose up -d"
    }
   stage('Slack Notification')
    {
             slackSend channel: '#yourchannel',
                    color: COLOR_MAP[currentBuild.getCurrentResult()],
                    tokenCredentialId:'slackCredentials',
                    message: "*${currentBuild.getCurrentResult()}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}"
    }
    }
    catch(e){
                slackSend channel: '#yourchannel',
                    color: 'danger',
                    tokenCredentialId:'slackCredentials',
                    message: "*ERROR:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n Description : ${e}\n More info at: ${env.BUILD_URL}"
    
    }
}