pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    options {
        // Keep build logs and fail fast on agent loss
        timeout(time: 60, unit: 'MINUTES')
        ansiColor('xterm')
    }

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Branch to build from')
        string(name: 'STUDENT_NAME', defaultValue: 'your name', description: 'Provide your name here, no name, no marks')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'prod'], description: 'Select environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run Jest tests after build')
    }

    environment {
        APP_VERSION = "1.0.${BUILD_NUMBER}"
        MAINTAINER = "Student"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out branch: ${params.BRANCH_NAME}"
                // Retry checkout a few times to reduce transient network errors
                retry(3) {
                    // use checkout scm so Jenkins uses job's configured repository and credentials
                    checkout scm
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Installing required packages..."
                // Capture npm errors and output package.json for debugging
                bat '''
                npm install || (
                  echo ======= npm install FAILED =======
                  echo package.json contents:
                  type package.json
                  echo ===================================
                  exit /b 1
                )
                '''
            }
        }

        stage('Build') {
            steps {
                echo "Building version ${APP_VERSION} for ${params.ENVIRONMENT} environment"
                bat '''
                echo Simulating build process...
                if not exist build mkdir build
                copy *.js build || echo "No .js files to copy"
                echo Build completed successfully!
                echo App version: %APP_VERSION% > build\\version.txt
                '''
            }
        }

        stage('Test') {
            when {
                expression { return params.RUN_TESTS }
            }
            steps {
                echo "Running Jest tests..."
                bat '''
                npm test || (
                  echo "Tests failed or jest not configured"
                  exit /b 1
                )
                '''
            }
        }

        stage('Package') {
            steps {
                echo "Creating zip archive for version ${APP_VERSION}"
                // triple-quoted bat to avoid Groovy parse errors and handle quoting reliably
                bat '''powershell -Command "Compress-Archive -Path 'build\\*' -DestinationPath \"build_${env:APP_VERSION}.zip\" -Force"'''
                // If zip fails, show files for debugging
                bat 'dir /s /b .'
            }
        }

        stage('Deploy (Simulation)') {
            steps {
                echo "Simulating deployment of version ${APP_VERSION} to ${params.ENVIRONMENT}"
            }
        }
    }

    post {
        always {
            node {
                echo "Cleaning up workspace..."
                deleteDir()
            }
        }
        success {
            echo "Pipeline succeeded! Version ${APP_VERSION} built and tested."
        }
        failure {
            echo "Pipeline failed! Check console output for details."
        }
    }
}



