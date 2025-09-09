pipeline {
    agent any

    environment {
        BRANCH_NAME = 'main'
        ECLIPSE_WORKSPACE = 'C:\\Users\\ankit\\eclipse-workspace\\AutomationExercise (2)\\AutomationExercise'
        COMMIT_MESSAGE = 'Jenkins: Auto-commit after build'
    }

    // Poll GitHub every hour at a random minute
    triggers {
        pollSCM('H * * * *')
    }

    stages {
        stage('Checkout from Git') {
            steps {
                echo 'Cloning repository from GitHub...'
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/Ankita2002-coder/AutomationExerciseAnkita.git'
            }
        }

        stage('Copy Files from Eclipse Workspace') {
            steps {
                bat """
                    echo Copying files from Eclipse workspace...
                    xcopy /E /Y /I "${ECLIPSE_WORKSPACE}\\*" "."
                """
            }
        }

        stage('Build & Test') {
            steps {
               
                bat 'mvn clean test'
                -DsuiteXmlFile = 'testng.xml'
            }
        }

        stage('Commit & Push Changes') {
            steps {
                script {
                    echo 'Checking for changes to commit and push...'

                    withCredentials([usernamePassword(
                        credentialsId: 'github-jenkins',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN')]) {

                        bat """
                            git config user.email "jenkins@automation.com"
                            git config user.name "Jenkins CI"

                            git status
                            git add .

                            REM Commit only if there are changes
                            git diff --cached --quiet || git commit -m "${COMMIT_MESSAGE}"

                            REM Push using token
                            git push https://%GIT_USER%:%GIT_TOKEN%@github.com/Ankita2002-coder/AutomationExerciseAnkita.git ${BRANCH_NAME}
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            // Publish Extent Report
            publishHTML(target: [
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'reports',
                reportFiles: 'ExtentReports.html',
                reportName: 'Extent Report'
            ])
        }

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo ' Pipeline failed. Please check logs and Extent Report.'
        }
    }
}
