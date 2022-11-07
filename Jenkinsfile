node {

	def err = null
	def buildVersion = "UNKNOWN"
	currentBuild.result = "SUCCESS"
	def buildStarted = false
	def timestamp = Calendar.getInstance().getTime().format('YYYYMMddHHmm',TimeZone.getTimeZone('GMT'))
	def buildDisplayName = "${env.JOB_NAME}".replaceAll('%2F','/');	

	try {
		def scriptState
		def branchname = "${env.BRANCH_NAME}"
		if (branchname == null || branchname == "null") {
		    branchname = "test-build"
		}
		branchname = "${branchname}".replaceAll("/","-")
		def versionState = "${branchname}".replaceAll("\\.","_")
				if (branchname == "master") {
			scriptState = ""
			branchname = "release"
		} else if (branchname == "develop") {
			scriptState = "beta"
			branchname = "beta"
		} else if (branchname == "release") {
			scriptState = "rc"
			branchname = "rc"
		}
		
		def branchLabel = "${branchname}".replaceAll("\\.","_")
		scriptState = "-${branchLabel}"
		echo "Building: ${buildDisplayName}"

		milestone()
		buildStarted = true;

		env.M2_HOME = tool 'Maven_3.6'
		stage('Checkout') {
			checkout scm
		}

		stage('Setup Build') {
			env.JAVA_HOME = tool 'OPEN_JDK_11'
			echo "JAVA_HOME: ${env.JAVA_HOME}"	
			properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '90', numToKeepStr: '')), disableConcurrentBuilds(abortPrevious: true)])
		}
		

		stage('Build') {
		    configFileProvider([configFile(fileId: 'DEFAULT_MAVEN_SETTINGS', variable: 'MAVEN_SETTINGS')]) {
				sh '$M2_HOME/bin/mvn -s $MAVEN_SETTINGS clean package deploy -f ./pom.xml -Dmaven.test.skip=true'
    		}
		}

		stage('Record Results') {
			recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), taskScanner(highTags: 'FIXME', ignoreCase: true, includePattern: '*/src/**/*.java', lowTags: 'XXX', normalTags: 'TODO')]
//			currentBuild.description="Version: ${buildVersion}"
		}

		stage('Archive Artifacts') {
			archiveArtifacts artifacts: '**/target/**/*.jar', followSymlinks: false
		}
		milestone()

	} catch (Exception ex) {
		echo "Build State: ${currentBuild.result}"
		
		if (buildStarted == true) {
			err = ex
			currentBuild.result = "FAILURE"
		}
		
	} finally {
		
//		if (currentBuild.result == "FAILURE" && buildStarted == true) {
//			emailext body: '''$BUILD_STATUS - $PROJECT_NAME - Build # $BUILD_NUMBER:
//
//Check console output at $BUILD_URL to view the results.
//
//Error Message: err.getMessage()''', recipientProviders: [brokenBuildSuspects(), upstreamDevelopers(), culprits(), requestor(), developers()], subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS!'	
//		
//		}

		if (err != null) {
		    throw err;
		}
	}

}