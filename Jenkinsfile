pipeline {
    agent any

    environment {
        BRANCH_NAME = 'main'
        COMMIT_MESSAGE = 'Jenkins: Auto-commit after build'
    }

    triggers {
        // Poll GitHub every hour
        pollSCM('H * * * *')
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from GitHub') {
            steps {
                echo 'Checking out repository...'
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                bat 'mvn clean test -DsuiteXmlFile=testng.xml'
            }
        }

        stage('Commit & Push Changes') {
            steps {
                script {
                    echo 'Checking for changes to commit and push...'

                    withCredentials([usernamePassword(
                        credentialsId: 'Github-jenkins',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN')]) {

                        bat """
                            git config user.email "jenkins@automation.com"
                            git config user.name "Jenkins CI"

                            git status
                            git add .

                            REM Commit only if there are changes
                            git diff --cached --quiet || git commit -m "${COMMIT_MESSAGE}"

                            REM Push using token, retry once if push fails
                            git push https://%GIT_USER%:%GIT_TOKEN%@github.com/Ankita2002-coder/AutomationExerciseAnkita.git ${BRANCH_NAME} || (
                                echo "Push failed, retrying..."
                                git push https://%GIT_USER%:%GIT_TOKEN%@github.com/Ankita2002-coder/AutomationExerciseAnkita.git ${BRANCH_NAME}
                            )
                        """
                    }
                }
            }
        }
    }

    post {
        always {
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
            echo 'Pipeline failed. Please check logs and Extent Report.'
        }
    }
}
