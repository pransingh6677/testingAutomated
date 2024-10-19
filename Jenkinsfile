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
        stage('Initialize') {
            steps {
                script {
                    // Initialize stage results as a map
                    // env.STAGE_RESULTS = groovy.json.JsonOutput.toJson([:])
                    // def stageResults = readJSON text: env.STAGE_RESULTS
                    // stageResults['Initialize'] = 'SUCCESS'
                    // env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
                }
            }
        }

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

  //       stage('SonarQube analysis') {
  //           steps {
  //               script {
  //                   try {
  //                       withSonarQubeEnv(installationName: 'SonarQube Scanner') {
  //                           sh """${scannerHome}/bin/sonar-scanner -X \
  //                               -D sonar.login=admin \
  //                               -D sonar.password=Proeffico@!234 \
  //                               -D sonar.projectBaseDir=/var/lib/jenkins/workspace/$JOB_NAME/ \
  //                               -D sonar.projectKey=Transact_AI \
  //                               -D sonar.sourceEncoding=UTF-8 \
  //                               -D sonar.host.url=http://sonarqube.proeffico.com/"""
  //                       }

  //                       def stageResults = readJSON text: env.STAGE_RESULTS
  //                       stageResults['SonarQube analysis'] = 'SUCCESS'
  //                       env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
  //                   } catch (Exception e) {
  //                       def stageResults = readJSON text: env.STAGE_RESULTS
  //                       stageResults['SonarQube analysis'] = 'FAILURE: ' + e.message
  //                       env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
  //                       throw e
  //                   }
  //               }
  //           }
  //       }

  //       stage('Quality gate') {
		//     steps {
		//       timeout(time: 1, unit: 'HOURS') {
		// 	waitForQualityGate abortPipeline: true
		//       }
		//     }
		// }

//         stage('SonarQube analysis') {
//     steps {
//         script {
//             try {
//                 withSonarQubeEnv(installationName: 'SonarQube Scanner') {
//                     sh """${scannerHome}/bin/sonar-scanner -X \
//                         -D sonar.login=admin \
//                         -D sonar.password=Proeffico@!234 \
//                         -D sonar.projectBaseDir=/var/lib/jenkins/workspace/$JOB_NAME/ \
//                         -D sonar.projectKey=Transact_AI \
//                         -D sonar.sourceEncoding=UTF-8 \
//                         -D sonar.host.url=http://sonarqube.proeffico.com/"""
//                 }

//                 def stageResults = readJSON text: env.STAGE_RESULTS
//                 stageResults['SonarQube analysis'] = 'SUCCESS'
//                 env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
//             } catch (Exception e) {
//                 def stageResults = readJSON text: env.STAGE_RESULTS
//                 stageResults['SonarQube analysis'] = 'FAILURE: ' + e.message
//                 env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
//                 throw e
//             }
//         }
//     }
// }

// stage('Quality gate') {
//     steps {
//         timeout(time: 1, unit: 'HOURS') {
//             script {
//                 def qualityGateResult = waitForQualityGate()
//                 if (qualityGateResult.status != 'OK') {
//                     def stageResults = readJSON text: env.STAGE_RESULTS
//                     stageResults['Quality gate'] = 'FAILURE: Quality Gate status is ' + qualityGateResult.status
//                     env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
//                     error("Quality Gate failed: ${qualityGateResult.status}")
//                 } else {
//                     def stageResults = readJSON text: env.STAGE_RESULTS
//                     stageResults['Quality gate'] = 'SUCCESS'
//                     env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
//                 }
//             }
//         }
//     }
// }


//         stage('Changing Ownership to Jenkins User') {
//             steps {
//                 script {
//                     try {
//                         sshPublisher(publishers: [sshPublisherDesc(
//                             configName: 'GKay',
//                             transfers: [sshTransfer(
//                                 cleanRemote: false,
//                                 excludes: '',
//                                 execCommand: 'sudo chown -R jenkins:jenkins /var/www/html/TransactAI_UI',
//                                 execTimeout: 120000,
//                                 flatten: false,
//                                 makeEmptyDirs: false,
//                                 noDefaultExcludes: false,
//                                 patternSeparator: '[, ]+',
//                                 remoteDirectory: '',
//                                 remoteDirectorySDF: false,
//                                 removePrefix: '',
//                                 sourceFiles: ''
//                             )],
//                             usePromotionTimestamp: false,
//                             useWorkspaceInPromotion: false,
//                             verbose: true
//                         )])

//                         def stageResults = readJSON text: env.STAGE_RESULTS
//                         stageResults['Changing Ownership to Jenkins User'] = 'SUCCESS'
//                         env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
//                     } catch (Exception e) {
//                         def stageResults = readJSON text: env.STAGE_RESULTS
//                         stageResults['Changing Ownership to Jenkins User'] = 'FAILURE: ' + e.message
//                         env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
//                         throw e
//                     }
//                 }
//             }
//         }

//         stage('Deploying') {
//             steps {
//                 script {
//                     try {
//                         // Run the deployment command on the remote server
//                         sshPublisher(publishers: [sshPublisherDesc(
//                             configName: 'GKay',
//                             transfers: [sshTransfer(
//                                 cleanRemote: false,
//                                 excludes: '',
//                                 execCommand: 'cd /var/www/html/TransactAI_UI/ && sudo bash deploy.sh; echo $? > /tmp/deploy_status.txt; cat /tmp/deploy_status.txt',
//                                 execTimeout: 1200000,
//                                 flatten: false,
//                                 makeEmptyDirs: false,
//                                 noDefaultExcludes: false,
//                                 patternSeparator: '[, ]+',
//                                 remoteDirectory: '/var/www/html/TransactAI_UI',
//                                 remoteDirectorySDF: false,
//                                 removePrefix: '',
//                                 sourceFiles: ''
//                             )],
//                             usePromotionTimestamp: false,
//                             useWorkspaceInPromotion: false,
//                             verbose: true
//                         )])

//                         // Update the stage results to SUCCESS
//                         def stageResults = readJSON text: env.STAGE_RESULTS
//                         stageResults['Deploying'] = 'SUCCESS'
//                         env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
//                     } catch (Exception e) {
//                         // Update the stage results to FAILURE and throw the exception
//                         def stageResults = readJSON text: env.STAGE_RESULTS
//                         stageResults['Deploying'] = 'FAILURE: ' + e.message
//                         env.STAGE_RESULTS = groovy.json.JsonOutput.toJson(stageResults)
//                         throw e
//                     }
//                 }
//             }
//         }
    }
           

    post {
        // success {
        //     script {
        //         def projectName = env.JOB_NAME
        //         def buildNumber = env.BUILD_NUMBER
        //         def buildUrl = env.BUILD_URL
        //         def commitAuthor = env.COMMIT_AUTHOR ?: 'Unknown'

        //         def stageResults = readJSON text: env.STAGE_RESULTS
        //         def stageResultsTable = """
        //         <table border="1" cellpadding="10" cellspacing="0" style="border-collapse: collapse; width: 100%;">
        //             <tr style="background-color: #196f3d; color: white;">
        //                 <th>Stage Name</th>
        //                 <th>Status</th>
        //             </tr>
        //         """
        //         stageResults.each { stage, result ->
        //             def rowColor = result.startsWith('SUCCESS') ? '#dff0d8' : '#f2dede'
        //             stageResultsTable += """
        //             <tr style="background-color: ${rowColor};">
        //                 <td>${stage}</td>
        //                 <td>${result}</td>
        //             </tr>
        //             """
        //         }
        //         stageResultsTable += "</table>"

        //         def emailBody = """
        //         <html>
        //         <body>
        //         <p>Hello,</p>
        //         <p>The pipeline for project <strong>'${projectName}'</strong> has completed successfully.</p>
        //         <p><strong>Build Number:</strong> ${buildNumber}</p>
        //         <p><strong>Build URL:</strong> <a href="${buildUrl}">${buildUrl}</a></p>
        //         <p><strong>Commit Author:</strong> <span style="color: #b91e08; font-weight: bold;">${commitAuthor}</span></p>
        //         <p><strong>Stage Results:</strong></p>
        //         ${stageResultsTable}
        //         <p>Please check the Jenkins console output for more details.</p>
        //         <p>Best regards,<br/>Proeffico_Jenkins</p>
        //         </body>
        //         </html>
        //         """

        //         def emails = env.COLLECTED_EMAILS.split(',').collect { it.trim() }
        //         emails.each { email ->
        //             emailext (
        //                 to: email,
        //                 from: 'jenkinsinfo@proeffico.com',
        //                 subject: "Pipeline Build Success for ${projectName} - Build #${buildNumber}",
        //                 body: emailBody,
        //                 mimeType: 'text/html'
        //             )
        //         }
        //     }
        // }

        failure {
            script {
                def emails = env.COLLECTED_EMAILS.split(',').collect { it.trim() }

                def projectName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def buildUrl = env.BUILD_URL
                def commitAuthor = env.COMMIT_AUTHOR ?: 'Unknown'

                def stageResults = readJSON text: env.STAGE_RESULTS
                def stageResultsTable = """
                <table border="1" cellpadding="10" cellspacing="0" style="border-collapse: collapse; width: 100%;">
                    <tr style="background-color: #f36f65; color: white;">
                        <th>Stage Name</th>
                        <th>Status</th>
                    </tr>
                """
                stageResults.each { stage, result ->
                    def rowColor = result.startsWith('SUCCESS') ? '#dff0d8' : '#f2958f'
                    stageResultsTable += """
                    <tr style="background-color: ${rowColor};">
                        <td>${stage}</td>
                        <td>${result}</td>
                    </tr>
                    """
                }
                stageResultsTable += "</table>"

                def emailBody = """
                <html>
                <body>
                <p>Hello,</p>
                <p>The pipeline for project <strong>'${projectName}'</strong> has failed.</p>
                <p><strong>Build Number:</strong> ${buildNumber}</p>
                <p><strong>Build URL:</strong> <a href="${buildUrl}">${buildUrl}</a></p>
                <p><strong>Commit Author:</strong> <span style="color: red; font-weight: bold;">${commitAuthor}</span></p>
                <p><strong>Stage Results:</strong></p>
                ${stageResultsTable}
                <p>Please check the Jenkins console output for more details.</p>
                <p>Best regards,<br/>Proeffico_Jenkins</p>
                </body>
                </html>
                """

                emails.each { email ->
                    emailext (
                        to: email,
                        from: 'jenkinsinfo@proeffico.com',
                        subject: "Pipeline Build Failure for ${projectName} - Build #${buildNumber}",
                        body: emailBody,
                        mimeType: 'text/html'
                    )
                }
            }
        }

        unstable {
            script {
                def emails = env.COLLECTED_EMAILS.split(',').collect { it.trim() }

                def projectName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def buildUrl = env.BUILD_URL
                def commitAuthor = env.COMMIT_AUTHOR ?: 'Unknown'

                def stageResults = readJSON text: env.STAGE_RESULTS
                def stageResultsTable = """
                <table border="1" cellpadding="10" cellspacing="0" style="border-collapse: collapse; width: 100%;">
                    <tr style="background-color: #f0e342; color: black;">
                        <th>Stage Name</th>
                        <th>Status</th>
                    </tr>
                """
                stageResults.each { stage, result ->
                    def rowColor = result.startsWith('SUCCESS') ? '#dff0d8' : '#f7c6c7'
                    stageResultsTable += """
                    <tr style="background-color: ${rowColor};">
                        <td>${stage}</td>
                        <td>${result}</td>
                    </tr>
                    """
                }
                stageResultsTable += "</table>"

                def emailBody = """
                <html>
                <body>
                <p>Hello,</p>
                <p>The pipeline for project <strong>'${projectName}'</strong> has completed with unstable status.</p>
                <p><strong>Build Number:</strong> ${buildNumber}</p>
                <p><strong>Build URL:</strong> <a href="${buildUrl}">${buildUrl}</a></p>
                <p><strong>Commit Author:</strong> <span style="color: orange; font-weight: bold;">${commitAuthor}</span></p>
                <p><strong>Stage Results:</strong></p>
                ${stageResultsTable}
                <p>Please check the Jenkins console output for more details.</p>
                <p>Best regards,<br/>Proeffico_Jenkins</p>
                </body>
                </html>
                """

                emails.each { email ->
                    emailext (
                        to: email,
                        from: 'jenkinsinfo@proeffico.com',
                        subject: "Pipeline Build Unstable for ${projectName} - Build #${buildNumber}",
                        body: emailBody,
                        mimeType: 'text/html'
                    )
                }
            }
        }
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
