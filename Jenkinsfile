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
                dir("${FOLDER_NAME}") {
                    deleteDir()
                    git branch: 'main', url: "${REPO_URL}"
                }
            }
        }

        stage('Set version') {
            steps {
                dir("${FOLDER_NAME}") {
                    writeFile file: 'version.txt', text: "${VERSION}"
                    echo "🔖 App version set to: ${VERSION}"
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
                    sleep 5 // даём серверу подняться
                    def raw = bat(
                        script: 'curl -s -o nul -w "%%{http_code}" http://localhost:5000',
                        returnStdout: true
                    ).trim()

                    def responseCode = raw.readLines().last().trim()
                    echo "🔎 Parsed response code: ${responseCode}"

                    if (responseCode != "200") {
                        error("❌ Health check failed with code: ${responseCode}")
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
                 subject: "✅ Build SUCCESS - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "The Jenkins job '${env.JOB_NAME}' build #${env.BUILD_NUMBER} finished successfully.\nVersion: ${VERSION}"
        }

        failure {
            echo '⚠️ Deployment failed.'
            dir("${FOLDER_NAME}") {
                bat 'docker-compose down || exit 0'
            }
            mail to: 'aigerim95.akk@gmail.com',
                 subject: "❌ Build FAILED - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "The Jenkins job '${env.JOB_NAME}' build #${env.BUILD_NUMBER} has failed."
        }
    }
}
