pipeline {
    agent any

    environment {
        CX_APIKEY = credentials('cx-api-key')  // Your Jenkins credential ID for API key
        CX_CLI_PATH = 'C:\\Rakshii\\cx-cli\\cx.exe'  // Path to your Checkmarx CLI executable
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/1.1']], 
                    userRemoteConfigs: [[url: 'https://github.com/Rakshhii/Exercises', credentialsId: 'rakshi']]
                ])
            }
        }

        stage('Configure CxOne CLI') {
            steps {
                withCredentials([string(credentialsId: 'cx-api-key', variable: 'CX_APIKEY')]) {
                    bat """
                        echo Configuring CxOne CLI...
                        ${env.CX_CLI_PATH} configure set --prop-name cx_base_uri --prop-value https://ind.ast.checkmarx.net
                        ${env.CX_CLI_PATH} configure set --prop-name cx_tenant --prop-value cx_ind_internal_test
                        echo API key will be passed via environment variable.
                    """
                }
            }
        }

        stage('Run CxOne Container Security Scan') {
            steps {
                withCredentials([string(credentialsId: 'cx-api-key', variable: 'CX_APIKEY')]) {
                    script {
                        echo 'Running container security scan...'

                        // Run scan and capture output
                        def scanOutput = bat(
                            script: """
                                set CX_APIKEY=%CX_APIKEY%
                                ${env.CX_CLI_PATH} scan create ^
                                    --project-name "Rakshhii/Exercises" ^
                                    --branch "1.1" ^
                                    -s .
                            """,
                            returnStdout: true
                        ).trim()

                        echo "Scan Output:\n${scanOutput}"

                        // Extract Scan ID from output for later stages
                        def scanIdMatch = scanOutput =~ /Scan ID\s*:\s*([a-f0-9-]+)/
                        if (!scanIdMatch) {
                            error("❌ Failed to extract Scan ID from scan output!")
                        }

                        env.SCAN_ID = scanIdMatch[0][1]
                        echo "Extracted Scan ID: ${env.SCAN_ID}"
                    }
                }
            }
        }

        stage('Check Results / Quality Gate') {
            steps {
                withCredentials([string(credentialsId: 'cx-api-key', variable: 'CX_APIKEY')]) {
                    script {
                        echo "Fetching results for Scan ID: ${env.SCAN_ID}"

                        def resultJson = bat(
                            script: """
                                set CX_APIKEY=%CX_APIKEY%
                                ${env.CX_CLI_PATH} results show --scan-id ${env.SCAN_ID} --format json
                            """,
                            returnStdout: true
                        ).trim()

                        echo "Scan Results JSON:\n${resultJson}"

                        // Simple check for HIGH or CRITICAL vulnerabilities
                        if (resultJson.contains('"HIGH"') || resultJson.contains('"CRITICAL"')) {
                            error("❌ High or Critical severity vulnerabilities found! Failing the pipeline.")
                        } else {
                            echo "✅ No high or critical severity vulnerabilities detected."
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished. Cleaning up workspace...'
            cleanWs()
        }
        failure {
            echo 'Build failed!'
        }
        success {
            echo 'Build succeeded!'
        }
    }
}
