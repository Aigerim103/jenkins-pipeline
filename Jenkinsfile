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
                    deleteDir() // очистка перед клоном
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
                    sleep 5 // wait for container to start
                    def response = bat(script: 'curl -s -o nul -w "%{http_code}" http://localhost:5001', returnStdout: true).trim()
                    if (response != "200") {
                        error("❌ Health check failed with response code: ${response}")
                    } else {
                        echo "✅ Health check passed!"
                    }
                }
            }
        }
    }
}
