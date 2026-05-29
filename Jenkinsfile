pipeline {

    agent any

    parameters {

        booleanParam(
            name: 'RUN_STABILITY',
            defaultValue: true,
            description: 'Run stability scan'
        )

        booleanParam(
            name: 'RUN_QUALITY',
            defaultValue: true,
            description: 'Run quality scan'
        )

        booleanParam(
            name: 'RUN_COVERAGE',
            defaultValue: true,
            description: 'Run coverage scan'
        )
    }

    stages {

        stage('Checkout') {

            steps {
                checkout scm
            }
        }

        stage('Parallel Scans') {

            parallel {

                stage('Code Stability') {

                    when {
                        expression {
                            params.RUN_STABILITY
                        }
                    }

                    steps {
                        sh 'mvn clean compile'
                    }
                }

                stage('Code Quality') {

                    when {
                        expression {
                            params.RUN_QUALITY
                        }
                    }

                    steps {
                        sh 'mvn checkstyle:checkstyle'
                    }
                }

                stage('Code Coverage') {

                    when {
                        expression {
                            params.RUN_COVERAGE
                        }
                    }

                    steps {
                        sh 'mvn test'
                    }
                }
            }
        }

        stage('Publish Reports') {

            steps {

		publishHTML(target: [
    		    allowMissing: false,
    		    alwaysLinkToLastBuild: true,
    		    keepAll: true,
    		    reportDir: 'target/site/jacoco',
    		    reportFiles: 'index.html',
    		    reportName: 'JaCoCo Coverage Report'
		])
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
            }
        }
    }

    post {

        success {
            echo 'Build Successful'
        }

        failure {
            echo 'Build Failed'
        }
    }
}
