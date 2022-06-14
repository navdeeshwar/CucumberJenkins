node() {

    def repoURL = 'https://github.com/navdeeshwar/CucumberJenkins.git'

    stage("Prepare Workspace") {
        cleanWs()
        env.WORKSPACE_LOCAL = sh(returnStdout: true, script: 'pwd').trim()
        env.BUILD_TIME = sh(returnStdout: true, script: 'date +%F-%T').trim()
        echo "Workspace set to:"
        echo "Build time:" + env.BUILD_TIME
    }
    stage('Checkout Self') {
        git branch: 'master', credentialsId: '', url: repoURL
    }
    stage('Cucumber Tests') {
         {
            sh """
			cd ${env.WORKSPACE_LOCAL}
			mvn clean test
		"""
        }
    }
    stage('Expose report') {
        archive "**/cucumber.json"
        cucumber '**/cucumber.json'
    }
	stage('Import results to Xray') {

		def description = "[BUILD_URL|${env.BUILD_URL}]"
		def labels = '["regression","automated_regression"]'
		
		def testExecutionFieldId = 10016
		def projectKey = "PROY"
		def xrayConnectorId = 'd6de8e3b-f0e8-4546-86de-33a8c6d23c7d'
		def info = '''{
				"fields": {
					"project": {
					"key": "''' + projectKey + '''"
				},
				"labels":''' + labels + ''',
				
				"summary": "Automated Regression Execution @ ''' + env.BUILD_TIME + ' ' + ''' " ,
				"issuetype": {
				"id": "''' + testExecutionFieldId + '''"
				},
				"''' + testEnvironmentFieldName + '''" 
				}
				}'''

			echo info

			step([$class: 'XrayImportBuilder', endpointName: '/cucumber/multipart', importFilePath: 'target/cucumber.json', importInfo: info, inputInfoSwitcher: 'fileContent', serverInstance: xrayConnectorId])
		}
}
