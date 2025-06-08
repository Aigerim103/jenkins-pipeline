pipeline {
    agent any

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['staging', 'production'], description: 'Select deployment environment')
    }

    environment {
        REPO_URL = 'https://github.com/Aigerim103/interactive-site.git'
        APP_NAME = 'interactive-site-web'
        VERSION = "v1.0-${params.ENVIRONMENT}-${new Date().format('dd-MM-yyyy-HHmm')}"
    }

    stages {
        stage('Clone project') {
            steps {
                dir('interactive-site') {
                    deleteDir()
                    git url: "${REPO_URL}", branch: 'main'
                }
            }
        }

        stage('Set version') {
            steps {
                dir('interactive-site') {
                    writeFile file: 'version.txt', text: "${VERSION}"
                    echo "🔖 App version set to: ${VERSION}"
                }
            }
        }

        stage('Build Docker image') {
            steps {
                dir('interactive-site') {
                    bat 'docker-compose build'
                }
            }
        }

        stage('Run containers') {
            steps {
                dir('interactive-site') {
                    bat 'docker-compose up -d'
                }
            }
        }

        stage('Health check') {
            steps {
                script {
                    sleep 5
                    def response = bat(script: 'curl -s -o /dev/null -w "%{http_code}" http://localhost:5000', returnStdout: true).trim()
                    echo "🔎 Parsed response code: ${response}"
                    if (response != '200') {
                        echo "❌ Health check failed. Rolling back..."
                        bat 'docker-compose down'
                        error("Health check failed. Deployment rolled back.")
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
                 body: "The deployment was successful.\n\nEnvironment: ${params.ENVIRONMENT}\nVersion: ${VERSION}"
        }
        failure {
            mail to: 'aigerim95.akk@gmail.com',
                 subject: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} — FAILED",
                 body: "The deployment failed. Please check the logs.\nEnvironment: ${params.ENVIRONMENT}"
        }
        always {
            dir('interactive-site') {
                bat 'docker-compose down || exit 0'
            }
        }
    }
}
