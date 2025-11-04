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
                                -Dsonar.javascript.enabled=false \
                                -Dsonar.typescript.enabled=false \
                                -Dsonar.css.enabled=false
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

        stage('DAST Scan with OWASP ZAP') {
            steps {
                echo "Iniciando escaneo DAST con ZAP..."
                sh '''
                    docker exec zap zap.sh -cmd -quickurl http://dvwa:80 -quickout /zap/wrk/report.html
                '''
            }
            post {
                always {
                    echo "Exportando reporte del escaneo"
                    sh 'docker cp zap:/zap/wrk/report.html .'
                    archiveArtifacts artifacts: 'report.html', fingerprint: true
                }
            }
        }
    }
}
