pipeline {
    agent any

    environment {
        APP_REPO = 'https://github.com/Aigerim103/interactive-site.git'
        VERSION = "v1.0-${BUILD_NUMBER}-${new Date().format('yyyyMMdd-HHmm')}"
    }

    stages {
        stage('Clone App Repository') {
            steps {
                dir('app') {
                    git "${APP_REPO}"
                }
            }
        }

        stage('Set Version') {
            steps {
                script {
                    writeFile file: 'app/version.txt', text: "${VERSION}"
                    echo "🔖 App version set to: ${VERSION}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('app') {
                    bat 'docker-compose build'
                }
            }
        }

        stage('Run Containers') {
            steps {
                dir('app') {
                    bat 'docker-compose up -d'
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    sleep 5
                    def response = bat(script: 'curl -s -o nul -w "%{http_code}" http://localhost:5000', returnStdout: true).trim()
                    if (response != "200") {
                        error("❌ Health check failed with code: ${response}")
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
                 subject: "✅ Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Application deployed successfully.\nVersion: ${VERSION}"
        }
        failure {
            echo '⚠️ Something went wrong. Rolling back.'
            dir('app') {
                bat 'docker-compose down || exit 0'
            }
            mail to: 'aigerim95.akk@gmail.com',
                 subject: "❌ Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "Build failed during pipeline execution."
        }
    }
}
