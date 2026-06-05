pipeline {

    agent any

    parameters {

        booleanParam(
            name: 'RUN_STABILITY',
            defaultValue: true,
            description: 'Run Stability Scan'
        )

        booleanParam(
            name: 'RUN_QUALITY',
            defaultValue: true,
            description: 'Run Quality Scan'
        )

        booleanParam(
            name: 'RUN_COVERAGE',
            defaultValue: true,
            description: 'Run Coverage Scan'
        )
    }

    environment {
        EMAIL_TO = 'sachin1420sm@gmail.com'
        SLACK_CHANNEL = '#jenkins-alerts'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Parallel Scans') {

            parallel {

                stage('Code Stability Analysis') {

                    when {
                        expression { params.RUN_STABILITY }
                    }

                    steps {

                        echo 'Running Stability Analysis'

                        sh '''
                            mvn compile
                        '''
                    }
                }

                stage('Code Quality Analysis') {

                    when {
                        expression { params.RUN_QUALITY }
                    }

                    steps {

                        echo 'Running Checkstyle Analysis'

                        sh '''
                            mvn checkstyle:checkstyle || true
                        '''
                    }
                }

                stage('Code Coverage Analysis') {

                    when {
                        expression { params.RUN_COVERAGE }
                    }

                    steps {

                        echo 'Running Unit Tests'

                        sh '''
                            mvn test
                        '''
                    }
                }
            }
        }

        stage('Approval') {

            steps {

                input(
                    message: 'Approve Artifact Publication?',
                    ok: 'Publish'
                )
            }
        }

        stage('Publish Artifact') {

            steps {

                sh 'mvn package'

                archiveArtifacts(
                    artifacts: 'target/*.jar',
                    fingerprint: true
                )

                echo 'Artifact Published Successfully'
            }
        }
    }

    post {

        success {

            emailext(
                to: "${EMAIL_TO}",
                subject: "SUCCESS : ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
Build Successful

Job Name: ${JOB_NAME}
Build Number: ${BUILD_NUMBER}

Build URL:
${BUILD_URL}
"""
            )

            slackSend(
                channel: "${SLACK_CHANNEL}",
                message: "SUCCESS : ${JOB_NAME} #${BUILD_NUMBER}\n${BUILD_URL}"
            )
        }

        failure {

            emailext(
                to: "${EMAIL_TO}",
                subject: "FAILED : ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
Build Failed

Job Name: ${JOB_NAME}
Build Number: ${BUILD_NUMBER}

Build URL:
${BUILD_URL}
"""
            )

            slackSend(
                channel: "${SLACK_CHANNEL}",
                message: "FAILED : ${JOB_NAME} #${BUILD_NUMBER}\n${BUILD_URL}"
            )
        }

        aborted {

            emailext(
                to: "${EMAIL_TO}",
                subject: "ABORTED : ${JOB_NAME} #${BUILD_NUMBER}",
                body: """
Artifact publication was rejected.

Job Name: ${JOB_NAME}
Build Number: ${BUILD_NUMBER}

Build URL:
${BUILD_URL}
"""
            )

            slackSend(
                channel: "${SLACK_CHANNEL}",
                message: "ABORTED : ${JOB_NAME} #${BUILD_NUMBER}\n${BUILD_URL}"
            )
        }
    }
}
