pipeline {

    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
        COURSE     = "Jenkins"
        // appVersion = ""
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
                sh 'npm install'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Sonar Scan') {
            steps {
                script {
                    def scannerHome = tool 'sonar-8.0'
                    withSonarQubeEnv('sonar-server') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

    //     stage('Build Image') {
    //         steps {
    //             script {
    //                 withAWS(region: 'us-east-1', credentials: 'aws-credentials') {
    //                     sh """
    //                         aws ecr get-login-password --region us-east-1 | \
    //                         docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com

    //                         docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
    //                         docker images
    //                         docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
    //                     """
    //                 }
    //             }
    //         }
    //     }
    // }

    post {
        always {
            echo 'I will always say Hello again!'
            cleanWs()
        }
        success {
            echo 'I will run if success'
        }
        failure {
            echo 'I will run if failure'
        }
        aborted {
            echo 'Pipeline is aborted'
        }
    }
}
