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

        stage('Run Check Login Tests') {
            steps {
                script {
                    catchError(
                        buildResult: 'FAILURE',
                        stageResult: 'FAILURE'
                    ) {
                        sh '''
                            echo "======================================"
                            echo " Running Check Login Tests"
                            echo "======================================"

                            bru run "Check login" \
                                --env Dev \
                                --reporter-junit reports/check-login-junit.xml \
                                --reporter-html reports/check-login-report.html

                            echo "======================================"
                            echo " Check Login Tests Completed"
                            echo "======================================"
                        '''
                    }
                }
            }
        }

        stage('Run Login Tests') {
            steps {
                script {
                    catchError(
                        buildResult: 'FAILURE',
                        stageResult: 'FAILURE'
                    ) {
                        sh '''
                            echo "======================================"
                            echo " Running Login Scenario Tests"
                            echo "======================================"

                            bru run "Login" \
                                --env Dev \
                                --reporter-junit reports/login-junit.xml \
                                --reporter-html reports/login-report.html

                            echo "======================================"
                            echo " Login Scenario Tests Completed"
                            echo "======================================"
                        '''
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
            echo " Test Reports Published"
            echo "======================================"
        }

        success {
            echo "======================================"
            echo " ALL LOGIN TESTS PASSED"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo " LOGIN TESTS FAILED"
            echo "======================================"

          
        }
    }
}