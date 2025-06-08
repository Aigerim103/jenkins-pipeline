pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Aigerim103/interactive-site.git'
        FOLDER_NAME = 'interactive-site'
        VERSION = "v1.0-${BUILD_NUMBER}-${new Date().format('yyyyMMdd-HHmm')}"
    }

    stages {
        stage('Clone project') {
            steps {
                bat "git clone ${REPO_URL} ${FOLDER_NAME}"
            }
        }

        stage('Set version') {
            steps {
                dir("${FOLDER_NAME}") {
                    script {
                        writeFile file: 'version.txt', text: "${VERSION}"
                        echo "🔖 App version set to: ${VERSION}"
                    }
                }
            }
        }

        stage('Build Docker image') {
            steps {
                dir("${FOLDER_NAME}") {
                    bat 'docker-compose build'
                }
            }
        }

        stage('Run containers') {
            steps {
                dir("${FOLDER_NAME}") {
                    bat 'docker-compose up -d'
                }
            }
        }

        stage('Health check') {
            steps {
                script {
                    sleep 5
                    def response = bat(script: 'curl -s -o nul -w "%{http_code}" http://localhost:5000', returnStdout: true).trim()
                    if (response != "200") {
                        error("❌ Health check failed with response code: ${response}")
                    } else {
                        echo "✅ Health check passed!"
                    }
                }
            }
        }
    }

    post {
        success {
            echo '🎉 Deployment successful!'
            mail to: 'aigerim95.akk@gmail.com',
                 subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Build successful!\n\nVersion: ${VERSION}\nJob: ${env.BUILD_URL}"
        }

        failure {
            echo '⚠️ Something went wrong.'
            dir("${FOLDER_NAME}") {
                bat 'docker-compose down || exit 0'
            }
            mail to: 'aigerim95.akk@gmail.com',
                 subject: "❌ FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Build failed!\n\nCheck the logs here: ${env.BUILD_URL}"
        }
    }
}
