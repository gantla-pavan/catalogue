pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
        COURSE    = "Jenkins"
        ACC_ID    = "515497299016"
        PROJECT   = "roboshop"
        COMPONENT = "catalogue"
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
                    env.appVersion = packageJSON.version
                    echo "app version: ${env.appVersion}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'npm test'
            }
        }

        // SonarQube Scan
        stage('Sonar Scan') {
            steps {
                script {
                    def scannerHome = tool 'sonar-8.0'
                    withSonarQubeEnv('sonar-scanner') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
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

        stage('Build Image') {
    steps {
        script {
            withAWS(region: 'us-east-1', credentials: 'aws-credentials') {
                sh """
                aws ecr get-login-password --region us-east-1 | \
                docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com

                docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${env.appVersion} .
                docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${env.appVersion}
                """
            }
        }
    }
}

                }
            }
        
    

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }

