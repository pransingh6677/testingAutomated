pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-Jenkins-token')
        CSV_API_URL = 'https://services.proeffico.com/api/downloadfile'
        CSV_API_TOKEN = credentials('csv-api-token-id')
        scannerHome = tool 'SonarQube Scanner'
    }
    
    parameters {
		// string(name: 'BRANCH_NAME', defaultValue: "", description: 'Branch to build')
		// choice(name: 'STAGE', choices: ['All', 'Build', 'Build-Scan-Upload'], description: 'Select the stage(s) to execute')

		choice choices: ['1', '2', '3'], description: 'Testing', name: 'BUMP_TYPE'
	}
  

	
    stages {


	    
        stage('Get Repo URL') {
            steps {
                script {
                    try {
                        def repoUrl = scm.getUserRemoteConfigs()[0].getUrl()
                        echo "Repository URL: ${repoUrl}"

                        def modifiedUrl = repoUrl.replace("github.com/", "api.github.com/repos/")
                        def indexGit = modifiedUrl.lastIndexOf(".git")
                        modifiedUrl = modifiedUrl.substring(0, indexGit) + "/collaborators" + modifiedUrl.substring(indexGit + ".git".length())

                        echo "Modified Repository URL: ${modifiedUrl}"

                        def response = sh(script: """
                            curl -H "Authorization: token ${GITHUB_TOKEN}" ${modifiedUrl}
                        """, returnStdout: true).trim()

                        echo "API Response: ${response}"

                        def json = readJSON text: response
                        def names = json.collect { it.login }.findAll { it != null }.join(',')
                        echo "Collected Names: ${names}"

                        env.COLLECTED_NAMES = names

                        def stageResults = readJSON text: env.STAGE_RESULTS
                        stageResults['Get Repo URL'] = 'SUCCESS'
                        env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
                    } catch (Exception e) {
                        def stageResults = readJSON text: env.STAGE_RESULTS
                        stageResults['Get Repo URL'] = 'FAILURE: ' + e.message
                        env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
                        throw e
                    }
                }
            }
        }

        stage('Fetch CSV File') {
            steps {
                script {
                    try {
                        sh(script: """
                           curl --location --request POST '${CSV_API_URL}' \
--header 'Authorization: Bearer ${CSV_API_TOKEN}' -o contact.csv
                        """)

                        echo "CSV file fetched."

                        def stageResults = readJSON text: env.STAGE_RESULTS
                        stageResults['Fetch CSV File'] = 'SUCCESS'
                        env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
                    } catch (Exception e) {
                        def stageResults = readJSON text: env.STAGE_RESULTS
                        stageResults['Fetch CSV File'] = 'FAILURE: ' + e.message
                        env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
                        throw e
                    }
                }
            }
        }

        stage('Extract Emails') {
            steps {
                script {
                    try {
                        def names = env.COLLECTED_NAMES.split(',').collect { it.trim() }
                        def emailAddresses = []

                        def csvData = readFile 'contact.csv'
                        def lines = csvData.split('\n')

                        lines.each { line ->
                            def columns = line.split(',')
                            if (columns.size() >= 2) {
                                def name = columns[0].trim()
                                def email = columns[1].trim()

                                if (names.contains(name)) {
                                    emailAddresses.add(email)
                                }
                            }
                        }

                        echo "Collected Email Addresses: ${emailAddresses.join(', ')}"
                        env.COLLECTED_EMAILS = emailAddresses.join(',')

                        def stageResults = readJSON text: env.STAGE_RESULTS
                        stageResults['Extract Emails'] = 'SUCCESS'
                        env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
                    } catch (Exception e) {
                        def stageResults = readJSON text: env.STAGE_RESULTS
                        stageResults['Extract Emails'] = 'FAILURE: ' + e.message
                        env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
                        throw e
                    }
                }
            }
        }

        stage('Get Commit Author') {
            steps {
                script {
                    try {
                        // Fetch the latest commit author using git command
                        def authorName = sh(script: 'git log -1 --pretty=format:%an', returnStdout: true).trim()
                        echo "Commit Author: ${authorName}"

                        // Store the author name in an environment variable
                        env.COMMIT_AUTHOR = authorName

                        def stageResults = readJSON text: env.STAGE_RESULTS
                        stageResults['Get Commit Author'] = 'SUCCESS'
                        env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
                    } catch (Exception e) {
                        def stageResults = readJSON text: env.STAGE_RESULTS
                        stageResults['Get Commit Author'] = 'FAILURE: ' + e.message
                        env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
                        throw e
                    }
                }
            }
        }

    }
           

    post {

                always {
            // sshPublisher(publishers: [sshPublisherDesc(
            //             configName: 'GKay',
            //             transfers: [sshTransfer(
            //                 cleanRemote: false,
            //                 excludes: '',
            //                 execCommand: 'sudo chown -R www-data:www-data /var/www/html/gkay-ui',
            //                 execTimeout: 120000,
            //                 flatten: false,
            //                 makeEmptyDirs: false,
            //                 noDefaultExcludes: false,
            //                 patternSeparator: '[, ]+',
            //                 remoteDirectory: '',
            //                 remoteDirectorySDF: false,
            //                 removePrefix: '',
            //                 sourceFiles: ''
            //             )],
            //             usePromotionTimestamp: false,
            //             useWorkspaceInPromotion: false,
            //             verbose: true // Set verbose to true for debugging
            //         )])
               // Delete workspace when build is done
            cleanWs(
                patterns: [[
                    pattern: 'dependency-check-report.xml',
                    exclude: true
                ]]
            ) // Jenkins Built-in-Feature for deleting the workspace and Cleaning env.
        }
    }
}
