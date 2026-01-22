pipeline {
    agent any

    environment {

        APP_VERSION = "1.0.${BUILD_NUMBER}"
        
        // Nazwa obrazu z wersją
        IMAGE_NAME = "moje-app:${APP_VERSION}"
        
        // Nazwa kontenera dla etapu Deploy
        DEPLOY_CONTAINER_NAME = "app-production-v${BUILD_NUMBER}"
        
        // Pliki wynikowe
        LOG_FILE = "build_log_${APP_VERSION}.txt"
        ARTIFACT_FILE = "aplikacja-v${APP_VERSION}.tar"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Builder') {
            steps {
                script {
                    echo "Budowanie obrazu (Build Container)"
                    sh "docker build -t ${IMAGE_NAME} . > ${LOG_FILE} 2>&1"
                }
            }
        }

        stage('Tester') {
            steps {
                script {
                    echo "Testy"
                    sh "docker run --rm ${IMAGE_NAME} npm test >> ${LOG_FILE} 2>&1"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Wdrażanie wersji ${APP_VERSION}"

                    sh "docker rm -f ${DEPLOY_CONTAINER_NAME} || true"
                    
                    sh "docker run -d --name ${DEPLOY_CONTAINER_NAME} -p 3000:3000 -e PORT=3000 ${IMAGE_NAME}"
                }
            }
        }


        stage('Smoke Test') {
            steps {
                script {
                    echo "Weryfikacja działania aplikacji"
                    sleep 5
                    
                    sh "curl -f -v http://localhost:3000 >> ${LOG_FILE} 2>&1"
                }
            }
        }

        stage('Publish') {
            steps {
                script {
                    echo "Publikacja Artefaktu"
                    sh "docker save ${IMAGE_NAME} -o ${ARTIFACT_FILE}"
                    
                    archiveArtifacts artifacts: "${ARTIFACT_FILE}", fingerprint: true
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "${LOG_FILE}", fingerprint: true

            sh "docker rm -f ${DEPLOY_CONTAINER_NAME} || true"
            sh "docker rmi ${IMAGE_NAME} || true"
        }
    }
}