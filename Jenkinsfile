
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
                        ],
                        [
                            name: 'MD-T19Enable SMS Two-Factor Authentication successfully',
                            report: 'md-t19'
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

                                    bru run "Login/${scenario.name}" --env Stage --reporter-junit reports/${scenario.report}-junit.xml --reporter-html reports/${scenario.report}-report.html

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
                        ],
                        [
                            name: 'MD-T46Incorrect OTPUnsuccessful Registration',
                            report: 'md-t46'
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

                                    bru run "Register/${scenario.name}" --env Stage --reporter-junit reports/${scenario.report}-junit.xml --reporter-html reports/${scenario.report}-report.html

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

        stage('Run Forgot Password Scenarios') {
            steps {
                script {

                    def scenarios = [
                        [
                            name: 'MD-T58Successful password reset with valid inputs',
                            report: 'md-t58'
                        ],
                        [
                            name: 'MD-T59Invalid or unregistered mobile number',
                            report: 'md-t59'
                        ],
                        [
                            name: 'MD-T60Incorrect OTP code',
                            report: 'md-t60'
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

                                    bru run "Forgot Password/${scenario.name}" --env Stage --reporter-junit reports/${scenario.report}-junit.xml --reporter-html reports/${scenario.report}-report.html

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

        stage('Run Invites Scenarios') {
            steps {
                script {

                    def scenarios = [
                        [
                            name: 'MD-T100Successfully invite a friend using a valid phone number',
                            report: 'md-t100'
                        ],
                        [
                            name: 'MD-T102Successfully cancel a sent invitation',
                            report: 'md-t102'
                        ],
                        [
                            name: 'MD-T103Re-invite a phone number that has already been invited',
                            report: 'md-t103'
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

                                    bru run "Invites/${scenario.name}" --env Stage --reporter-junit reports/${scenario.report}-junit.xml --reporter-html reports/${scenario.report}-report.html

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

        stage('Run Action Post Scenarios') {
            steps {
                script {

                    def scenarios = [
                        [
                            name: 'MD-T4Like & Dislike a post',
                            report: 'md-t4'
                        ],
                        [
                            name: 'MD-T5 Add a text comment on a post & Delete',
                            report: 'md-t5'
                        ],
                        [
                            name: 'MD-T10Repost a post',
                            report: 'md-t10'
                        ],
                        [
                            name: 'MD-T11Repost -Quote a post',
                            report: 'md-t11'
                        ],
                        [
                            name: 'MD-T185Successfully bookmark a post',
                            report: 'md-t18'
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

                                    bru run "Action Post/${scenario.name}" --env Stage --reporter-junit reports/${scenario.report}-junit.xml --reporter-html reports/${scenario.report}-report.html

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
           stage('Run Post Scenarios') {
            steps {
                script {

                    def scenarios = [
                        [
                            name: 'MD-T31Delete own post successfully',
                            report: 'md-t31'
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

                                    bru run "Post/${scenario.name}" --env Stage --reporter-junit reports/${scenario.report}-junit.xml --reporter-html reports/${scenario.report}-report.html

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
