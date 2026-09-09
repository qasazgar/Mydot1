pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Environment') {
            steps {
                sh ''
                '
                echo "======================================"
                echo "Node version:"
                node--version

                echo "NPM version:"
                npm--version

                echo "Bruno version:"
                bru--version

                echo "======================================"
                ''
                '
            }
        }

        stage('Prepare Reports') {
            steps {
                sh ''
                '
                rm - rf reports
                rm - rf temp - reports
                rm - rf test - logs

                mkdir - p reports
                mkdir - p temp - reports
                mkdir - p test - logs ''
                '
            }
        }

        stage('Run All Scenarios') {
            steps {
                script {

                    def defaultEnv = 'SuperApp-dev-BDD'

                    def scenarios = [

                        // =====================================================
                        // 01 - End To End (Run with SuperApp-dev)
                        // =====================================================
                        [
                            name: 'Check login',
                            path: 'Check login',
                            env: 'Stage'
                        ],

                        // =====================================================
                        // 02 - Login (Run with SuperApp-dev-BDD)
                        // =====================================================
                        [
                            name: 'Login',
                            path: 'MD-T38Login with a valid phone number and incorrect password'
                            env: 'Stage'

                        ],
                        [
                            name: 'Login',
                            path: 'MD-T39Login with a valid username and incorrect password'
                            env: 'Stage'
                        ],
                        [
                            name: 'Login',
                            path: 'MD-T40Login using OTP with a phone number'
                            env: 'Stage'
                        ]

                    ]

                    // =========================================================
                    // Test Execution
                    // =========================================================
                    def failedTests = []

                    echo ""
                    echo "=============================================="
                    echo "TOTAL SCENARIOS: ${scenarios.size()}"
                    echo "=============================================="

                    for (scenario in scenarios) {

                        def targetEnv = scenario.env ? : defaultEnv

                        echo ""
                        echo "=============================================="
                        echo "Running Scenario: ${scenario.name}"
                        echo "Path: ${scenario.path}"
                        echo "Environment: ${targetEnv}"
                        echo "=============================================="

                        def junitFile = "temp-reports/${scenario.name}-junit.xml"
                        def htmlFile = "reports/${scenario.name}-report.html"
                        def logFile = "test-logs/${scenario.name}.log"

                        def result = sh(
                            script: ""
                            "#!/bin/bash
                            set + e set - o pipefail

                            bru run "${scenario.path}"\\
                            --env "${targetEnv}"\\
                            --reporter - junit "${junitFile}"\\
                            --reporter - html "${htmlFile}"\\
                            2 > & 1 | tee "${logFile}"

                            EXIT_CODE = \$ ?

                            echo ""
                            echo "Bruno Exit Code: \$EXIT_CODE"

                            exit\ $EXIT_CODE ""
                            ",
                            returnStatus : true
                        )

                        if (result != 0) {
                            failedTests.add(scenario.name)
                            echo ""
                            echo "❌ FAILED: ${scenario.name}"
                            echo "Exit Code: ${result}"
                        } else {
                            echo ""
                            echo "✅ PASSED: ${scenario.name}"
                        }

                        echo ""
                    }

                    // =========================================================
                    // Save Failed Tests
                    // =========================================================
                    writeFile(
                        file: 'reports/failed-tests.txt',
                        text: failedTests.join('\n')
                    )

                    // =========================================================
                    // Test Execution Summary
                    // =========================================================
                    def totalTests = scenarios.size()
                    def failedCount = failedTests.size()
                    def passedCount = totalTests - failedCount

                    echo ""
                    echo "=============================================="
                    echo "TEST EXECUTION SUMMARY"
                    echo "=============================================="
                    echo "Total Scenarios : ${totalTests}"
                    echo "Passed          : ${passedCount}"
                    echo "Failed          : ${failedCount}"
                    echo "=============================================="

                    if (failedCount > 0) {
                        echo ""
                        echo "Failed Scenarios:"
                        echo "----------------------------------------------"
                        failedTests.each {
                            echo "❌ ${it}"
                        }
                        echo "----------------------------------------------"
                        currentBuild.result = 'UNSTABLE'
                    } else {
                        echo ""
                        echo "🎉 ALL SCENARIOS PASSED"
                    }

                    echo "=============================================="
                }
            }
        }
    }

    // ========================================================================
    // POST ACTIONS
    // ========================================================================
    post {

        always {
            echo ""
            echo "Publishing Jenkins JUnit Test Reports..."

            junit(
                testResults: 'temp-reports/*-junit.xml',
                allowEmptyResults: true,
                skipPublishingChecks: false
            )

            archiveArtifacts(
                artifacts: 'reports/**/*.html, reports/failed-tests.txt, test-logs/**/*.log',
                allowEmptyArchive: true,
                fingerprint: true
            )
        }

        unstable {
            echo ""
            echo "=============================================="
            echo "⚠️ TESTS FAILED (BUILD UNSTABLE)"
            echo "=============================================="

            script {
                if (fileExists('reports/failed-tests.txt')) {
                    def failed = readFile('reports/failed-tests.txt').trim()
                    if (failed) {
                        echo ""
                        echo "Failed scenarios detail:"
                        echo "----------------------------------------------"
                        echo failed
                        echo "----------------------------------------------"
                    }
                }
            }
        }

        success {
            echo ""
            echo "=============================================="
            echo "✅ ALL TESTS PASSED"
            echo "=============================================="
        }

        failure {
            echo ""
            echo "=============================================="
            echo "❌ PIPELINE FAILED"
            echo "=============================================="
        }

        cleanup {
            echo ""
            echo "=============================================="
            echo "Jenkins Test Execution Completed"
            echo "=============================================="
        }
    }
}