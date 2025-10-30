pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube' 
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    withCredentials([string(credentialsId: 'tokenpipe', variable: 'SONAR_AUTH_TOKEN')]) {
                        script {
                            def scannerHome = tool 'SonarQubeScanner'
                            sh """
                                ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=testPipeLine \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://10.30.212.100:9000 \
                                -Dsonar.token=$SONAR_AUTH_TOKEN \
                                -Dsonar.javascript.file.suffixes=-
                            """
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
