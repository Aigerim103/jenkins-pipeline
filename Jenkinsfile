pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['staging', 'production'],
            defaultValue: 'staging',
            description: 'Select deployment environment'
        )
    }

    environment {
        REPO_URL = 'https://github.com/Aigerim103/interactive-site.git'
        FOLDER_NAME = 'interactive-site'
        VERSION = "v1.0-${params.ENVIRONMENT ?: 'staging'}-${new Date().format('yyyyMMdd-HHmm')}"
    }

    stages {
        stage('Clone project') {
            steps {
                dir("${FOLDER_NAME}") {
                    deleteDir()
                    git url: "${REPO_URL}", branch: 'main'
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
                subject: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} — SUCCESS",
                body: """The deployment was successful 🎉

Environment: ${params.ENVIRONMENT}
Version: ${VERSION}
Job: ${env.JOB_NAME} #${env.BUILD_NUMBER}
"""
        }

        failure {
            echo '⚠️ Deployment failed.'
            mail to: 'aigerim95.akk@gmail.com',
                subject: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} — FAILED",
                body: """The deployment failed ❌

Environment: ${params.ENVIRONMENT}
Version: ${VERSION}
Please check the Jenkins logs for details.
"""
        }

        always {
            echo '🧹 Cleaning up containers...'
            dir("${FOLDER_NAME}") {
                bat 'docker-compose down || exit 0'
            }
        }
    }
}
