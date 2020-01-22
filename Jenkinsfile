pipeline
{
    agent any
	tools
	{
		maven 'Maven'
	}
	options
    {
        // Append time stamp to the console output.
        timestamps()
      
        timeout(time: 1, unit: 'HOURS')
      
        // Do not automatically checkout the SCM on every stage. We stash what
        // we need to save time.
        skipDefaultCheckout()
      
        // Discard old builds after 10 days or 30 builds count.
        buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))
	  
	    //To avoid concurrent builds to avoid multiple checkouts
	    disableConcurrentBuilds()
    }
    stages
    {
	    stage ('checkout')
		{
			steps
			{
				checkout scm
			}
		}
		stage ('Build')
		{
			steps
			{
				bat "mvn install"
			}
		}
		stage ('Unit Testing')
		{
			steps
			{
				bat "mvn test"
			}
		}
		// stage ('Sonar Analysis')
		// {
		// 	steps
		// 	{
		// 		withSonarQubeEnv("Test_Sonar") 
		// 		{
		// 			bat "mvn sonar:sonar"
		// 		}
		// 	}
		// }
		// stage ('Upload to Artifactory')
		// {
		// 	steps
		// 	{
		// 		rtMavenDeployer (
    //                 id: 'deployer',
    //                 serverId: '123456789@artifactory',
    //                 releaseRepo: 'CI-Automation-JAVA',
    //                 snapshotRepo: 'CI-Automation-JAVA'
    //             )
    //             rtMavenRun (
    //                 pom: 'pom.xml',
    //                 goals: 'clean install',
    //                 deployerId: 'deployer',
    //             )
    //             rtPublishBuildInfo (
    //                 serverId: '123456789@artifactory',
    //             )
		// 	}
		// }
		stage ('Docker Image')
		{
			steps
			{
				bat returnStdout: true, script: 'docker build -t devopssampleapplication_tarungarg:%BUILD_NUMBER% -f Dockerfile .'
			}
		}
		//stage ('Push to DTR')
	    //{
		//    steps
		  //  {
		    //	bat returnStdout: true, script: 'docker push devopssampleapplication_tarungarg:%BUILD_NUMBER%'
		    //}
	    //}
      stage ('Stop Running container')
    	{
	        steps
	        {
	            bat '''
                    	@echo off
			FOR /f "tokens=*" %%i IN ('docker ps -a') DO docker stop %%i && docker rm %%i
			FOR /f "tokens=*" %%i IN ('docker images --format "{{.ID}}"') DO docker rmi %%i
                '''
	        }
	    }

		stage ('Docker deployment')
		{
		    steps
		    {
		        bat 'docker run --name devopssampleapplication_tarungarg -d -p 5016:8080 devopssampleapplication_tarungarg:%BUILD_NUMBER%'
		    }
		}
	}
	post 
	{
        always 
		{
			emailext attachmentsPattern: 'report.html', body: '${JELLY_SCRIPT,template="health"}', mimeType: 'text/html', recipientProviders: [[$class: 'RequesterRecipientProvider']], replyTo: 'tarun.garg@nagarro.com', subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!', to: 'tarun.garg@nagarro.com'
        }
    }
}
