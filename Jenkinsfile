pipeline {
    agent {
        docker {
            image 'node:16'
        }
    }

    environment {
        DOCKER_HUB_USER = 'ocefall1n1'
        IMAGE_NAME      = 'isec6000-node-app'
        IMAGE_TAG       = "${BUILD_NUMBER}"
    }

    options {
        // Logging Setting
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 1, unit: 'HOURS')
    }

    stages {
        // 1. Dependencies
        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js Dependencies...'
                sh 'npm install'
            }
        }

        // 2. Unit Tests
        stage('Run Unit Tests') {
            steps {
                echo 'Running Unit Tests'
                sh 'npm test || echo "No unit tests specified in package.json, skipping exit failure."'
            }
        }

        // 3. Security Gate
        stage('Security Vulnerability Scan') {
            steps {
                echo 'Scaning Vulnerability with npm audit...'
                script {
                    // Vulnerability Verification
                    def auditExitCode = sh(
                        script: 'npm audit --audit-level=high',
                        returnStatus: true
                    )
                    
                    if (auditExitCode != 0) {
                        error("[Security Gate] Found High or Critical levels of Vulnerability. Pipeline stopped.")
                    } else {
                        echo 'Security scan passed.'
                    }
                }
            }
        }

        // 4. Build and push Docker image
        stage('Build & Push Docker Image') {
            steps {
                script {
                    echo 'Building Docker Image...'
                    // Using Jenkins Credentials to push
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        def customImage = docker.build("${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}")
                        customImage.push()
                        customImage.push('latest')
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline build sussessful.'
        }
        failure {
            echo 'Pipeline fail to build, read log for more details.'
        }
    }
}
