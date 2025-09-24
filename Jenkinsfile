pipeline {
    agent any

    environment {
        CX_TENANT = 'cx_ind_internal_test'
        CX_BASE_URI = 'https://ind.ast.checkmarx.net'
        CX_CLI_PATH = 'C:\\Rakshii\\cx-cli\\cx'  // Path to cx.exe
    }

    stages {
        stage('Configure CxOne CLI') {
            steps {
                withCredentials([string(credentialsId: 'cx-api-key', variable: 'CX_APIKEY')]) {
                    bat """
                        echo Configuring CxOne CLI...
                        %CX_CLI_PATH% configure set --prop-name cx_base_uri --prop-value %CX_BASE_URI%
                        %CX_CLI_PATH% configure set --prop-name cx_tenant --prop-value %CX_TENANT%
                        %CX_CLI_PATH% configure set --prop-name cx_api_key --prop-value %CX_APIKEY%
                    """
                }
            }
        }

        stage('Run CxOne Container Security Scan') {
            steps {
                bat """
                    echo Running container security scan...
                    %CX_CLI_PATH% scan create ^
                        --project-name "Rakshhii/Exercises" ^
                        --branch "1.1" ^
                        -s .
                """
            }
        }

        stage('Check Results / Quality Gate') {
            steps {
                script {
                    def result = bat(script: '%CX_CLI_PATH% results show --last --format json', returnStdout: true).trim()
                    echo "Scan Results: ${result}"

                    if (result.contains('"HIGH"')) {
                        error("❌ High severity vulnerabilities found! Failing the pipeline.")
                    } else {
                        echo "✅ No high severity vulnerabilities detected. Pipeline passes."
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Cleaning up workspace..."
            cleanWs()
        }
    }
}
