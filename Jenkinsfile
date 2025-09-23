pipeline {
    agent any  // ✅ Ensures workspace is allocated and FilePath context exists

    environment {
        CX_APIKEY = credentials('eyJhbGciOiJIUzUxMiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICIzNTM0MTE1MC0yNjFlLTRkZDctYmU0YS1kMzAzYmY4YzcwM2YifQ.eyJpYXQiOjE3NTg2NDA5OTYsImp0aSI6ImY5OWIwMWRmLWU5YWItNDU5Zi1iMDY0LWViOTc4NTFhZjk4ZSIsImlzcyI6Imh0dHBzOi8vaW5kLmlhbS5jaGVja21hcngubmV0L2F1dGgvcmVhbG1zL2N4X2luZF9pbnRlcm5hbF90ZXN0IiwiYXVkIjoiaHR0cHM6Ly9pbmQuaWFtLmNoZWNrbWFyeC5uZXQvYXV0aC9yZWFsbXMvY3hfaW5kX2ludGVybmFsX3Rlc3QiLCJzdWIiOiJjMzA2YTVhNi1jZTBlLTRkOTMtOTk5Ny00NTYzNzAzNDlkNzIiLCJ0eXAiOiJPZmZsaW5lIiwiYXpwIjoiYXN0LWFwcCIsInNpZCI6ImYyNjlkNmM5LWRjOGMtNDY2Yy04Nzg1LWQxOGYxZDQ4ZTRjZiIsInNjb3BlIjoicHJvZmlsZSBpYW0tYXBpIGdyb3VwcyByb2xlcyBlbWFpbCBhc3QtYXBpIG9mZmxpbmVfYWNjZXNzIn0.zUrzyVZU4QRRGWD2NolBcoG0a7w3huhJLlZCtmCGy8SJYZhW39c1igX1QXxAhPwhMLVuJccdZ9rbJzMrBfoZtg')  // Jenkins secret text ID
        CX_TENANT = 'cx_ind_internal_test'           // Change to your tenant
        CX_BASE_URI = 'https://ind.ast.checkmarx.net' // Or your CxOne URL
    }

    stages {
        stage('Configure CxOne CLI') {
            steps {
                sh """
                    echo "Configuring CxOne CLI..."
                    cx configure set base-uri $CX_BASE_URI
                    cx configure set tenant $CX_TENANT
                    cx configure set api-key $CX_APIKEY
                """
            }
        }

        stage('Run CxOne Container Security Scan') {
            steps {
                sh """
                    echo "Running container security scan..."
                    cx scan create \
                        --project-name "Rakshhii/Exercises" \
                        --branch "1.1" \
                        -s .
                """
            }
        }

        stage('Check Results / Quality Gate') {
            steps {
                script {
                    def result = sh(script: "cx results show --last --format json", returnStdout: true).trim()
                    echo "Scan Results: ${result}"

                    // Simple quality gate check for HIGH severity findings
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
