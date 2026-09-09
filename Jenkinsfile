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
                sh '''
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
                    rm -rf temp-reports
                    rm -rf test-logs

                    mkdir -p reports
                    mkdir -p temp-reports
                    mkdir -p test-logs
                '''
            }
        }

        stage('Run All Scenarios') {
            steps {
                script {

                    def defaultEnv = 'SuperApp-dev-BDD'

                    def scenarios = [

                        [
                            name: 'Check login',
                            path: 'Check login',
                            env: 'Stage'
                        ],

                        [
                            name: 'MD-T38 Login with valid phone and incorrect password',
                            path: 'MD-T38Login with a valid phone number and incorrect password',
                            env: 'Stage'
                        ],

                        [
                            name: 'MD-T39 Login with valid username and incorrect password',
                            path: 'MD-T39Login with a valid username and incorrect password',
                            env: 'Stage'
                        ],

                        [
                            name: 'MD-T40 Login using OTP with phone number',
                            path: 'MD-T40Login using OTP with a phone number',
                            env: 'Stage'
                        ]
                    ]

                    def failedTests = []

                    echo "=============================================="
                    echo "TOTAL SCENARIOS: ${scenarios.size()}"
                    echo "=============================================="

                    for (scenario in scenarios) {

                        def targetEnv = scenario.env ?: defaultEnv

                        def safeName = scenario.name.replaceAll(
                            /[^a-zA-Z0-9_-]/,
                            '_'
                        )

                        def junitFile = "temp-reports/${safeName}-junit.xml"
                        def htmlFile = "reports/${safeName}-report.html"
                        def logFile = "test-logs/${safeName}.log"

                        echo ""
                        echo "=============================================="
                        echo "Running Scenario: ${scenario.name}"
                        echo "Path: ${scenario.path}"
                        echo "Environment: ${targetEnv}"
                        echo "=============================================="

                        withEnv([
                            "SCENARIO_PATH=${scenario.path}",
                            "TARGET_ENV=${targetEnv}",
                            "JUNIT_FILE=${junitFile}",
                            "HTML_FILE=${htmlFile}",
                            "LOG_FILE=${logFile}"
                        ]) {

                            def result = sh(
                                script: '''
                                    #!/bin/bash

                                    set +e
                                    set -o pipefail

                                    echo "Running Bruno..."
                                    echo "Scenario: $SCENARIO_PATH"
                                    echo "Environment: $TARGET_ENV"

                                    bru run "$SCENARIO_PATH" \
                                        --env "$TARGET_ENV" \
                                        --reporter junit "$JUNIT_FILE" \
                                        --reporter html "$HTML_FILE" \
                                        2>&1 | tee "$LOG_FILE"

                                    EXIT_CODE=$?

                                    echo ""
                                    echo "======================================"
                                    echo "Bruno Exit Code: $EXIT_CODE"
                                    echo "======================================"

                                    exit $EXIT_CODE
                                ''',
                                returnStatus: true
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
                        }
                    }

                    writeFile(
                        file: 'reports/failed-tests.txt',
                        text: failedTests.join('\n')
                    )

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
                }
            }
        }
    }

    post {

        always {

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

            echo "=============================================="
            echo "⚠️ TESTS FAILED (BUILD UNSTABLE)"
            echo "=============================================="

            script {

                if (fileExists('reports/failed-tests.txt')) {

                    def failed = readFile(
                        'reports/failed-tests.txt'
                    ).trim()

                    if (failed) {
                        echo ""
                        echo "Failed scenarios:"
                        echo "----------------------------------------------"
                        echo failed
                        echo "----------------------------------------------"
                    }
                }
            }
        }

        success {

            echo "=============================================="
            echo "✅ ALL TESTS PASSED"
            echo "=============================================="
        }

        failure {

            echo "=============================================="
            echo "❌ PIPELINE FAILED"
            echo "=============================================="
        }

        cleanup {

            echo "=============================================="
            echo "Jenkins Test Execution Completed"
            echo "=============================================="
        }
    }
}