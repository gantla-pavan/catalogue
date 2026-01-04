pipeline {

    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
        COURSE     = "Jenkins"
        appVersion = ""
        ACC_ID     = "515497299016"
        PROJECT    = "roboshop"
        COMPONENT  = "catalogue"
    }

    options {
        timeout(time: 10, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {

        stage('Read Version') {
            steps {
                script {
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "App version: ${appVersion}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                  npm install
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                  npm test
                '''
            }
        }

        stage('Sonar Scan') {
            environment {
                scannerHome = tool 'sonar-8.0'
            }
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                      ${scannerHome}/bin/sonar-scanner \
                      -Dsonar.projectKey=${PROJECT}-${COMPONENT} \
                      -Dsonar.projectName=${PROJECT}-${COMPONENT} \
                      -Dsonar.sources=. \
                      -Dsonar.exclusions=node_modules/** \
                      -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // stage('Dependabot Security Gate') {
        //     environment {
        //         GITHUB_OWNER = 'daws-86s'
        //         GITHUB_REPO  = 'catalogue'
        //         GITHUB_API   = 'https://api.github.com'
        //         GITHUB_TOKEN = credentials('GITHUB_TOKEN')
        //     }

        //     steps {
        //         sh '''
        //           echo "Fetching Dependabot alerts..."

        //           response=$(curl -s \
        //             -H "Authorization: token ${GITHUB_TOKEN}" \
        //             -H "Accept: application/vnd.github+json" \
        //             "${GITHUB_API}/repos/${GITHUB_OWNER}/${GITHUB_REPO}/dependabot/alerts?per_page=100")

        //           echo "${response}" > dependabot_alerts.json

        //           high_critical_open_count=$(echo "${response}" | jq '[.[] 
        //             | select(
        //                 .state == "open"
        //                 and (.security_advisory.severity == "high"
        //                      or .security_advisory.severity == "critical")
        //             )
        //           ] | length')

        //           echo "Open HIGH/CRITICAL Dependabot alerts: ${high_critical_open_count}"

        //           if [ "${high_critical_open_count}" -gt 0 ]; then
        //               echo "❌ Blocking pipeline due to OPEN HIGH/CRITICAL Dependabot alerts"
        //               echo "${response}" | jq '.[] 
        //                 | select(.state=="open" 
        //                 and (.security_advisory.severity=="high" 
        //                 or .security_advisory.severity=="critical"))
        //                 | {dependency: .dependency.package.name, severity: .security_advisory.severity, advisory: .security_advisory.summary}'
        //               exit 1
        //           else
        //               echo "✅ No HIGH/CRITICAL Dependabot alerts found"
        //           fi
        //         '''
            }
        }

//     }
// }
