    pipeline {
        agent any

        environment {
            IMAGE_NAME = "testowa-aplikacja:${BUILD_NUMBER}"
            LOG_FILE = "log_procesu_build_${BUILD_NUMBER}.txt"
        }

        stages {
            stage('Checkout') {
                steps {
                    checkout scm
                }
            }

            stage('Build inside Container') {
                steps {
                    script {
                        echo "KROK 1: Budowanie obrazu"
                        sh "docker build -t ${IMAGE_NAME} . > ${LOG_FILE} 2>&1"
                    }
                }
            }

            stage('Test inside Container') {
                steps {
                    script {
                        echo "KROK 2: Testowanie"
                        sh "docker run --rm ${IMAGE_NAME} npm test >> ${LOG_FILE} 2>&1"
                    }
                }
            }
        }

        post {
            always {
                archiveArtifacts artifacts: "${LOG_FILE}", fingerprint: true
                
                sh "docker rmi ${IMAGE_NAME} || true"
            }
        }
    }