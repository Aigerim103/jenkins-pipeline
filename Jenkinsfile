pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Aigerim103/interactive-site.git'
        FOLDER_NAME = 'interactive-site'
        VERSION = "v1.0-${BUILD_NUMBER}-${new Date().format('yyyyMMdd-HHmm')}"
        SLACK_WEBHOOK_URL = credentials('slack-webhook') // Предварительно добавить в Jenkins > Manage Credentials
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
                    sleep 5
                    def raw = bat(
                        script: 'curl -s -o nul -w "%%{http_code}" http://localhost:5000',
                        returnStdout: true
                    ).trim()

                    echo "🩺 Raw response: ${raw}"

                    // Оставляем только "200"
                    def responseCode = raw.replaceAll('[^0-9]', '')
                    echo "✅ Parsed code: ${responseCode}"

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
                 body: "Jenkins job '${env.JOB_NAME}' build #${env.BUILD_NUMBER} completed successfully.\nVersion: ${VERSION}"

            slackSend (
                color: 'good',
                message: "✅ *Build SUCCESS* — ${env.JOB_NAME} #${env.BUILD_NUMBER}\nVersion: ${VERSION}",
                webhookUrl: "${SLACK_WEBHOOK_URL}"
            )
        }

        failure {
            echo '⚠️ Deployment failed.'
            dir("${FOLDER_NAME}") {
                bat 'docker-compose down || exit 0'
            }
            mail to: 'aigerim95.akk@gmail.com',
                 subject: "❌ Build FAILED - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Build failed. Check logs in Jenkins."

            slackSend (
                color: 'danger',
                message: "❌ *Build FAILED* — ${env.JOB_NAME} #${env.BUILD_NUMBER}\nPlease check the Jenkins console log.",
                webhookUrl: "${SLACK_WEBHOOK_URL}"
            )
        }
    }
}
