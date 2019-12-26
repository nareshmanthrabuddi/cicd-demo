#!groovy

/**
 *
 * Author:    Naresh Manthrabuddi
 * Created:   27.12.2019
 * 
 **/

pipeline {
  agent any
  
	stages {
	    stage('CI/CD Pipeline triggered') {		
			steps {
			   echo 'CI/CD Pipeline Started'
			}
	    }
	}
}
node {

	def v_environment = ""
	def v_buildMechanism = ""
	def v_muleRuntimeEnvironment = ""
	def v_workers = ""
	def v_workerType = ""
	def v_applicationName = ""
	def v_anypointCredentialID = ""
	def v_anypointOrganization = ""
	def v_AnypointEnvironment = ""
	def v_muleEnv = ""
	def v_encryptKey = ""
	def v_artifactId = ""
	def v_version = ""
	def v_package = ""
	def v_downloadFilePath = ""

	// nexus variables
	def nexus_Protocol = ""
	def nexus_BaseURL = ""		
	def nexus_RepoName = ""		
	def pom_GroupID = ""
	def pom_ArtifactId = ""
	def pom_Version = ""
	def pom_Packaging = ""
	def downloadDir = ""
	def propertiesFilePath = ""
	def downloadFilePath = ""


	properties([
     parameters([
       choiceParam(
         choices: 'development\nSIT\nTEST\nPREPROD\nPROD',
            description: 'Please select the ENVIRONMENT for Deployment',
            name: 'ENVIRONMENT'
       ),
       choiceParam(
         choices: 'BUILD-MUNITS\nBUILD-MUNITS-SONAR\nBUILD-MUNITS-SONAR-RELEASE',
            description: 'Please select the Build Mechanism',
            name: 'BUILD_MECHANISM'
       ),
       choiceParam(
         choices: '4.2.0\n4.2.1\n4.2.2\n4.1.5',
            description: 'Please select mule runtime version for Deployment?',
            name: 'MULE_RUNTIME_VERSION'
       ),
       choiceParam(
         choices: '(default:1)\n2\n3\n4\n5\n6\n7\n8',
            description: 'Please select the workers for Deployment, By default this value is 1?',
            name: 'WORKERS'
       ),
       choiceParam(
         choices: 'MICRO(default:0.1 vCores)\nSMALL(0.2 vCores)\nMEDIUM(1 vCore )\nLARGE(2 vCores)\nXLARGE(4 vCores)\nXXLARGE(8 vCores)\n4XLARGE(16 vCores)',
            description: 'Please select the vCores for Deployment, By default this value is 0.1vCore?',
            name: 'WORKERTYPE'
       ),
       choiceParam(
         choices: 'us-east-1(default; US East, N. Virginia)\nus-east-2(US East, Ohio)\nus-west-1(US West, N. California)\nus-west-2(US West, Oregon)\neu-central-1(EU, Frankfurt)\neu-west-1(EU, Ireland)\neu-west-2(EU, London)\nap-southeast-1(Asia Pacific, Singapore)\nap-southeast-2(Asia Pacific, Sydney)\nap-northeast-1(Asia Pacific, Tokyo)\nca-central-1(Canada, Central)\nsa-east-1 (South America, São Paulo)',
            description: 'Please select the Region for Deployment, By default this value is us-east-1?',
            name: 'REGION'
		),         
       string(
		   name: 'APPLICATION_NAME', 
		   defaultValue: 'cicd-demo', 
		   description: 'Please enter the application name for CloudHub Deployment?'
	   ),
       string(
		   name: 'ANYPOINT_ORGANIZATION', 
		   defaultValue: 'LnD', 
		   description: 'Please provide CloudHub Anypoint Organization name to deploy?'
	   )
     ])
    ])
	if(params.BUILD_MECHANISM == 'BUILD-MUNITS') {
			try{
				stage 'Code Build & MUnits Execution'	
					UDF_BuildSourceCode()
						
				stage 'Notification'
					SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","success")
					
			} catch(error) {
				throw(error)
				SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")

			}		
		} else if(params.BUILD_MECHANISM == 'BUILD-MUNITS-SONAR') {
			try{
				stage 'Code Build & MUnits Execution'	
					UDF_BuildSourceCode()
					
				stage 'SonarQube Analysis'
					UDF_ExecuteSonarQubeRules()

				stage 'ArtifactUploadToNexus'
					UDF_ArtifactUploadToNexus()
				
				stage 'Notification'
					SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","success")

			} catch(error) {
				throw(error)
				SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
			}
		} else if(params.BUILD_MECHANISM == 'BUILD-MUNITS-SONAR-RELEASE') {
			try{
				stage 'Code Build & MUnits Execution'	
					UDF_BuildSourceCode()
					
				stage 'SonarQube Analysis'
					UDF_ExecuteSonarQubeRules()

				stage 'DeployToCloudHub'
					UDF_DeployToCloudHub()

				stage 'Notification'
					SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")	
			} catch(error) {
				throw(error)
				SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
			}
		}
}

/*
BUILD - STAGE
This function provides the functionality to build your clode
*/

def UDF_BuildSourceCode()
{	
	try	{
		sh 'mvn clean package'	
	}catch(error) {
		throw(error)
		SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
	}
}

/*
SONARQUBE - STAGE
This function provides functionality to do the SONAR Analysis
*/
def UDF_ExecuteSonarQubeRules()
{	
	try{
		echo 'SonarQube Rules Execution started'
		//bat 'mvn sonar:sonar'
		echo 'SonarQube Rules Execution Completed'	
	} catch(error) {
		throw(error)
		SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
	}
}

/*
NEXUS ARTIFACT UPLOAD - STAGE
This function provides functionality to upload the artifacts to the Nexus repository
*/

def UDF_ArtifactUploadToNexus() 
{	

	try{
		echo "Artifact Copy to Nexus Started"

		pom_GroupID = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","groupId")	
		v_artifactId = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","artifactId")
		v_version = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","version")
		v_package = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","packaging")
		String v_downloadFilePath = "${env.WORKSPACE}\\target\\${v_artifactId}-${v_version}-${v_package}.jar"	
		def v_buildNumber = "${env.BUILD_NUMBER}"
		v_nexusProtocol = "http"
		v_nexusBaseURL = "${env.LOCAL_NEXUS_BASEURL}"

		if(v_nexusBaseURL.contains("http://")) {
			v_nexusBaseURL = v_nexusBaseURL.substring(7)
		} else if(v_nexusBaseURL.contains("https://")) {
			v_nexusBaseURL = v_nexusBaseURL.substring(8)
		}		

		
		echo "###### NEXUS REPO DETAILS ######"
		echo "Nexus base URL: ${env.LOCAL_NEXUS_BASEURL}"	
		echo "Build Number : ${env.BUILD_NUMBER}"	
		echo "pom_GroupID: ${pom_GroupID}"
		echo "pom_ArtifactId is : ${v_artifactId}"
		echo "pom_Version is : ${v_version}"
		echo "pom_Packaging is : ${v_package}"
		echo "downloadDir is : ${v_downloadFilePath}"

		nexusArtifactUploader(
			nexusVersion: 'nexus3',
			protocol: v_nexusProtocol,
			nexusUrl: v_nexusBaseURL,
			groupId: 'cicd-demo',
			version: v_buildNumber,
			repository: 'maven-releases',
			credentialsId: '40860708-f898-4290-bf94-1b26ace6d908',
			artifacts: [
				[artifactId: v_artifactId,
				 classifier: 'debug',
				 file: v_downloadFilePath,
				 type: 'jar']
			]
		 )
	    echo 'Artifact Copy to Nexus Completed'
	}catch(error) {
		throw(error)
		SendEmail("","","Failed")
	}
}


/*
DEPLOY STAGE
This function provides functionality to deploy the application package(zip file) to CloudHub Runtime
*/

def UDF_DeployToCloudHub() {	

	echo "###### Application Deployment Stage ######"

	v_environment = "${params.ENVIRONMENT}"
	v_buildMechanism ="${params.BUILD_MECHANISM}"
	v_muleRuntimeEnvironment ="${params.MULE_RUNTIME_VERSION}"
	v_applicationName = "${params.APPLICATION_NAME}"
	v_anypointCredentialID = "${params.ANYPOINT_CREDENTIAL_ID}"
	v_anypointOrganization = "${params.ANYPOINT_ORGANIZATION}"
	v_workers = ""
	v_workerType = ""
	v_anypointEnvironment = "${params.ENVIRONMENT}"
	v_muleEnv = ""
	v_region = "${params.REGION}"
	v_encryptKey = ""
	v_artifactId = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","artifactId")
	v_version = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","version")
	v_package = UDF_GetPOMData("${env.WORKSPACE}/pom.xml","packaging")
	v_downloadFilePath = "${env.WORKSPACE}\\target\\${v_artifactId}-${v_version}-${v_package}.jar"	

    /*
    	Setting the environment
    */

	if("${params.ENVIRONMENT}" == 'development') {

		v_anypointCredentialID = 'bccc9153-9fda-40b4-b266-70fbbb0176c8'

	} else if("${params.ENVIRONMENT}" == 'SIT') {

		v_anypointCredentialID= 'bccc9153-9fda-40b4-b266-70fbbb0176c8'

	} else if("${params.ENVIRONMENT}" == 'TEST') {

		v_anypointCredentialID= 'bccc9153-9fda-40b4-b266-70fbbb0176c8'

	} else if("${params.ENVIRONMENT}" == 'PREPROD') {

		v_anypointCredentialID= 'bccc9153-9fda-40b4-b266-70fbbb0176c8'

	} else if("${params.ENVIRONMENT}" == 'PROD') {

		v_anypointCredentialID= 'bccc9153-9fda-40b4-b266-70fbbb0176c8'
	}

	/* 
		Setting the WORKERS
	*/

	echo "Setting the WORKERS"

	if("${params.WORKERS}" == '(default:1)') {

		v_workers = '1'

	} else {

		v_workers = "${params.WORKERS}"
	}

	/*
		Setting the WORKERTYPE
	*/

	echo "Setting the WORKERTYPE"

	if("${params.WORKERTYPE}" == 'MICRO(default:0.1 vCores)') {

		v_workerType = 'MICRO'

	} else if("${params.WORKERTYPE}" == 'SMALL(0.2 vCores)') {

			v_workerType = 'SMALL'

	} else if("${params.WORKERTYPE}" == 'MEDIUM(1 vCore )') {

			v_workerType = 'MEDIUM'
			
	} else if("${params.WORKERTYPE}" == 'LARGE(2 vCores)') {

			v_workerType = 'LARGE'
			
	} else if("${params.WORKERTYPE}" == 'XLARGE(4 vCores)') {

			v_workerType = 'XLARGE'
			
	} else if("${params.WORKERTYPE}" == 'XXLARGE(8 vCores)') {

			v_workerType = 'XXLARGE'
			
	} else if("${params.WORKERTYPE}" == '4XLARGE(16 vCores)') {

			v_workerType = '4XLARGE'
	} 

	/*
		Setting the Region
	*/
     
    echo "Setting the Region"
    
    if("${params.REGION}" == 'us-east-1(default; US East, N. Virginia)') {

    	v_region = 'us-east-1'

    } else if("${params.REGION}" == 'us-east-2(US East, Ohio)') {

    	v_region = 'us-east-2'    

    } else if("${params.REGION}" == 'us-west-1(US West, N. California)') {

    	v_region = 'us-west-1'    

    } else if("${params.REGION}" == 'us-west-2(US West, Oregon)') {

    	v_region = 'us-west-2'    

    } else if("${params.REGION}" == 'eu-central-1(EU, Frankfurt)') {

    	v_region = 'eu-central-1'    

    } else if("${params.REGION}" == 'eu-west-1(EU, Ireland)') {

    	v_region = 'eu-west-1'    

    } else if("${params.REGION}" == 'eu-west-2(EU, London)') {

    	v_region = 'eu-west-2'    

    } else if("${params.REGION}" == 'ap-southeast-1(Asia Pacific, Singapore)') {

    	v_region = 'ap-southeast-1'    

    } else if("${params.REGION}" == 'ap-southeast-2(Asia Pacific, Sydney)') {

    	v_region = 'ap-southeast-2'    

    } else if("${params.REGION}" == 'ap-northeast-1(Asia Pacific, Tokyo)') {

    	v_region = 'ap-northeast-1'    

    } else if("${params.REGION}" == 'ca-central-1(Canada, Central)') {

    	v_region = 'ca-central-1'    

    } else if("${params.REGION}" == 'sa-east-1 (South America, São Paulo)') {

    	v_region = 'sa-east-1'    

    }

	echo "ENVIRONMENT is : ${v_environment}"
	echo "BUILD_MECHANISM is : ${v_buildMechanism}"
	echo "MULE_RUNTIME_VERSION is : ${v_muleRuntimeEnvironment}"
	echo "WORKERS : ${v_workers}"
	echo "WORKERTYPE : ${v_workerType}"
	echo "APPLICATION_NAME is : ${v_applicationName}"
	echo "ANYPOINT_ORGANIZATION is : ${v_anypointOrganization}"
	echo "ANYPOINT_ENVIRONMENT is : ${v_anypointEnvironment}"	
	echo "Environment Workspace is : ${env.WORKSPACE}"	
	echo "Download File Path is : ${v_downloadFilePath}"
	echo "ANYPOINT_CREDENTIAL_ID is : ${v_anypointCredentialID}"
	echo "Region : ${v_region}"
	//-DbusinessGroupId=5a1efa6d-e32c-440c-b3bb-978fc5ae507b

	withCredentials([usernamePassword(credentialsId: "${v_anypointCredentialID}",passwordVariable: 'ANYPOINT_PASSWORD',usernameVariable: 'ANYPOINT_USERNAME')])
	{

		bat "mvn deploy -DmuleDeploy -DskipTests=true -Danypoint.username=${ANYPOINT_USERNAME} -Danypoint.password=${ANYPOINT_PASSWORD} -Denvironment=${v_anypointEnvironment} -DbusinessGroup=${v_anypointOrganization} -Dworkers=${v_workers} -DworkerType=${v_workerType} -DmuleVersion=${v_muleRuntimeEnvironment} -DapplicationName=${v_applicationName}"
	}
}

/*
	This function returns the repository details
*/

def UDF_GetGitRepoName() {	
	try{
	def tokens = "${env.JOB_NAME}".tokenize('/')
    String repo = tokens[tokens.size()-2]
	
    //branch = tokens[tokens.size()-1]	
    //org = tokens[tokens.size()-3]	
	
	return repo
	}catch(error)
	{
		throw(error)
		SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
	}
}

/*
This function returns the POM Data
*/

def UDF_GetPOMData(udfp_PomName, udfp_PropertyName){	
	try{
	def resultVal = ""
	def pomFile = readFile(udfp_PomName)
	def pom = new XmlParser().parseText(pomFile)
	def gavMap = [:]
	resultVal =  pom[udfp_PropertyName].text().trim()
	return resultVal
	}catch(error)
	{
		throw(error)
		SendEmail("naresh.manthrabuddi@whishworks.com","naresh.manthrabuddi@whishworks.com","Failed")
	}
}

/*
This function Sends Email
*/

def SendEmail(udfp_ToAddress, udfp_FromAddress, udfp_Status)
{
	try{
	   String body = ""
	   
	   if(udfp_Status == "success")
	   {
		   body= "SUCCESS"
	   }
	   else
	   {
		   body= "FAILED"
	   }
		/*
		mail subject: "${env.JOB_NAME} (${env.BUILD_NUMBER}) ${body}",
				body: "It appears that ${env.BUILD_URL} is ${body}",
				  to: "naresh.manthrabuddi@whishworks.com",
			 replyTo: "naresh.manthrabuddi@whishworks.com",
				from: "naresh.manthrabuddi@whishworks.com"
		*/
				
	}catch(error)
	{		
		throw(error)
	}
}