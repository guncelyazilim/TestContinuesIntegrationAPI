//Kaynak kodun içerisindeki projenin ismi
String projectName = "TestContinuesIntegrationAPI\\TestContinuesIntegrationAPI"

//Kaynak kodun publish edileceği dizin
String publishedPath = "TestContinuesIntegrationAPI\\TestContinuesIntegrationAPI\\bin\\Release\\netcoreapp3.1\\publish"

//Hedef makinesindeki IIS'de tanımlı olan sitenizin ismi
String iisApplicationName = "TestWebApp"

//Hedef makinesindeki IIS'de tanımlı olan sitenizin dizini
String iisApplicationPath = "C:\\inetpub\wwwroot\\ReactVideoApp"

//Hedef makinesinin IP'si
String targetServerIP = "10.10.16.10"

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
		  bat "dotnet build TestContinuesIntegrationAPI\\TestContinuesIntegrationAPI.csproj --configuration Release /p:Version=${BUILD_NUMBER}"
		
    }
	stage('Publish'){
       bat "dotnet publish TestContinuesIntegrationAPI\\TestContinuesIntegrationAPI.csproj "
     
	}
	stage('Deploy'){
        withCredentials([usernamePassword(credentialsId: 'iis-credential', usernameVariable: 'GUNCEL\seren.ogur', passwordVariable: 'guncel@123')]) { bat """ "C:\\Program Files (x86)\\IIS\\Microsoft Web Deploy V3\\msdeploy.exe" -verb:sync -source:iisApp="${WORKSPACE}\\${publishedPath}" -enableRule:AppOffline -dest:iisApp="${iisApplicationName}",ComputerName="https://${targetServerIP}:8172/msdeploy.axd",UserName="$USERNAME",Password="$PASSWORD",AuthType="Basic" -allowUntrusted"""}
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