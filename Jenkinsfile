pipeline {
    agent any

    environment {
        CX_TENANT = 'cx_ind_internal_test'
        CX_BASE_URI = 'https://ind.ast.checkmarx.net'
    }

    stages {
        stage('Configure CxOne CLI') {
            steps {
                withCredentials([string(credentialsId: 'cx-api-key', variable: 'CX_APIKEY')]) {
                    bat """
                        echo Configuring CxOne CLI...
                        C:\\Rakshii\\cx-cli\\cx configure set base-uri %CX_BASE_URI%
                        C:\\Rakshii\\cx-cli\\cx configure set tenant %CX_TENANT%
                        C:\\Rakshii\\cx-cli\\cx configure set api-key %CX_APIKEY%
                    """
                }
            }
        }

        stage('Run CxOne Container Security Scan') {
            steps {
                bat """
                    echo Running container security scan...
                    C:\\Rakshii\\cx-cli\\cx scan create ^
                        --project-name "Rakshhii/Exercises" ^
                        --branch "1.1" ^
                        -s .
                """
            }
        }

        stage('Check Results / Quality Gate') {
            steps {
                script {
                    def result = bat(script: 'C:\\Rakshii\\cx-cli\\cx results show --last --format json', returnStdout: true).trim()
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
