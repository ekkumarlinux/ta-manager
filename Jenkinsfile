pipeline {
    agent any

    parameters {
        stashedFile(name: 'REPORT', description: 'Upload the Excel file')
        string(name: 'Account_IDs', defaultValue: '', description: 'Enter the Account Numbers (comma-separated)')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the code from the GitHub repository
                    dir("Report-Manager") {
                        git branch: 'main', url: 'https://github.com/ekkumarlinux/ta-manager.git'
                    }
                }
            }
        }

        stage('Create Reports Directory') {
            steps {
                script {
                    // Create 'reports' directory if it doesn't exist
                    sh 'mkdir -p reports'
                }
            }
        }

        stage('Copy Report File') {
            steps {
                script {
                    // Unstash the uploaded file and maintain its original name
                    def originalFileName = sh(returnStdout: true, script: 'basename $REPORT')
                    sh "mv $REPORT reports/$originalFileName"
                }
            }
        }

        stage('Install Required Modules') {
            steps {
                script {
                    // Check if requirements.txt exists and install required modules
                    def requirementsFileExists = fileExists('requirements.txt')
                    if (requirementsFileExists) {
                        sh 'pip install -r requirements.txt'
                    } else {
                        echo "No requirements.txt file found. Skipping module installation."
                    }
                }
            }
        }

        stage('Process Report File') {
            steps {
                script {
                    def accountIDs = params.Account_IDs
                    echo "Processing report with account IDs: $accountIDs"

                    // Check if the 'reports' directory exists
                    def reportsDirExists = fileExists('reports')
                    if (!reportsDirExists) {
                        error "The 'reports' directory does not exist."
                    }

                    // Check if the report file exists
                    def originalFileName = sh(returnStdout: true, script: 'basename $REPORT')
                    def reportFileExists = fileExists("reports/$originalFileName")
                    if (!reportFileExists) {
                        error "The report file '$originalFileName' does not exist."
                    }

                    // Run your Python script to process the report file
                    sh "python3 main.py reports/$originalFileName $accountIDs"
                }
            }
        }

        stage('Archive Processed File') {
            steps {
                script {
                    // Archive the processed Excel file for download
                    archiveArtifacts artifacts: "reports/*", onlyIfSuccessful: true
                }
            }
        }
    }

    post {
        always {
            // Clean up the workspace by deleting the processed files
            cleanWs()
        }
    }
}
