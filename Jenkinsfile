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
                                --env Stage \
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
                        ],
                        [
                            name: 'MD-T34Login with a valid username',
                            report: 'md-t34'
                        ],
                        [
                            name: 'MD-T36Login with a Invalid username',
                            report: 'md-t36'
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

                                    bru run "Login/${scenario.name}" \
                                        --env Stage \
                                        --reporter-junit reports/${scenario.report}-junit.xml \
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

        stage('Run Register Scenarios') {
            steps {
                script {

                    def scenarios = [
                        [
                            name: 'MD-T45Successful Registration',
                            report: 'md-t45'
                        ],
                        [
                            name: 'MD-T53Existing User Login Navigation',
                            report: 'md-t53'
                        ],
                        [
                            name: 'MD-T51Duplicate Email',
                            report: 'md-t51'
                        ],
                        [
                            name: 'MD-T51Duplicate Username',
                            report: 'md-t511'
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

                                    bru run "Register/${scenario.name}" \
                                        --env Stage \
                                        --reporter-junit reports/${scenario.report}-junit.xml \
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
            echo " ALL TESTS PASSED"
            echo "======================================"
        }

        failure {

            echo "======================================"
            echo " SOME TESTS FAILED"
            echo "======================================"
        }
    }
}