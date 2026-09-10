pipeline {
    agent any

    options {
        disableConcurrentBuilds()

        buildDiscarder(
            logRotator(
                numToKeepStr: '20',
                artifactNumToKeepStr: '10'
            )
        )
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Environment') {
            steps {
                sh '''
                    echo "======================================"
                    echo " Environment Check"
                    echo "======================================"

                    echo "Node version:"
                    node --version

                    echo "NPM version:"
                    npm --version

                    echo "Bruno version:"
                    bru --version

                    echo "======================================"
                '''
            }
        }

        stage('Prepare Reports') {
            steps {
                sh '''
                    rm -rf reports
                    mkdir -p reports
                '''
            }
        }

        stage('Run Check Login') {
            steps {
                script {
                    catchError(
                        buildResult: 'FAILURE',
                        stageResult: 'FAILURE'
                    ) {
                        sh '''
                            echo "======================================"
                            echo " Running Check Login"
                            echo "======================================"

                            bru run "Check login" \
                                --env Dev \
                                --reporter-junit reports/check-login-junit.xml \
                                --reporter-html reports/check-login-report.html

                            echo "======================================"
                            echo " Check Login Completed"
                            echo "======================================"
                        '''
                    }
                }
            }
        }

        stage('Run Login Scenarios') {
            steps {
                script {

                    def scenarios = [
                        [
                            name: 'MD-T38Login with a valid phone number and incorrect password',
                            report: 'md-t38'
                        ],
                        [
                            name: 'MD-T39Login with a valid username and incorrect password',
                            report: 'md-t39'
                        ],
                        [
                            name: 'MD-T40Login using OTP with a phone number',
                            report: 'md-t40'
                        ]
                    ]

                    for (scenario in scenarios) {

                        stage("Run ${scenario.report.toUpperCase()}") {

                            catchError(
                                buildResult: 'FAILURE',
                                stageResult: 'FAILURE'
                            ) {

                                sh """
                                    echo "======================================"
                                    echo " Running: ${scenario.name}"
                                    echo "======================================"

                                    bru run "Login/${scenario.name}" \\
                                        --env Dev \\
                                        --reporter-junit reports/${scenario.report}-junit.xml \\
                                        --reporter-html reports/${scenario.report}-report.html

                                    echo "======================================"
                                    echo " Completed: ${scenario.name}"
                                    echo "======================================"
                                """
                            }
                        }
                    }
                }
            }
        }
    }

    post {

        always {

            echo "======================================"
            echo " Publishing Test Results"
            echo "======================================"

            junit(
                allowEmptyResults: true,
                testResults: 'reports/*-junit.xml'
            )

            archiveArtifacts(
                artifacts: 'reports/*.html',
                allowEmptyArchive: true
            )

            echo "======================================"
            echo " All Reports Published"
            echo "======================================"
        }

        success {

            echo "======================================"
            echo " ALL LOGIN TESTS PASSED"
            echo " No SMS will be sent."
            echo "======================================"
        }

        failure {

            echo "======================================"
            echo " LOGIN TESTS FAILED"
            echo " Running SMS Notification..."
            echo "======================================"

            sh '''
                bru run "sms" || true
            '''

            echo "======================================"
            echo " SMS Notification Completed"
            echo "======================================"
        }
    }
}