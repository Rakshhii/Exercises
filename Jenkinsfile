stage('Check Results / Quality Gate') {
    steps {
        withCredentials([string(credentialsId: 'cx-api-key', variable: 'CX_APIKEY')]) {
            script {
                // Capture scan ID from previous stage (scan create output)
                def scanOutput = bat(
                    script: """
                        set CX_APIKEY=%CX_APIKEY%
                        %CX_CLI_PATH% scan create ^
                            --project-name "Rakshhii/Exercises" ^
                            --branch "1.1" ^
                            -s .
                    """,
                    returnStdout: true
                ).trim()

                // Extract Scan ID using regex
                def scanIdMatch = scanOutput =~ /Scan ID\s*:\s*([a-f0-9-]+)/
                if (!scanIdMatch) {
                    error("❌ Failed to extract Scan ID.")
                }

                def scanId = scanIdMatch[0][1]
                echo "🔍 Extracted Scan ID: ${scanId}"

                // Run results show
                def resultJson = bat(
                    script: """
                        set CX_APIKEY=%CX_APIKEY%
                        %CX_CLI_PATH% results show --scan-id ${scanId} --format json
                    """,
                    returnStdout: true
                ).trim()

                echo "Scan Results (JSON): ${resultJson}"

                // Check for "HIGH" severity vulnerabilities
                if (resultJson.contains('"HIGH"') || resultJson.contains('"CRITICAL"')) {
                    error("❌ High or Critical severity vulnerabilities found! Failing the pipeline.")
                } else {
                    echo "✅ No high or critical severity vulnerabilities detected."
                }
            }
        }
    }
}
