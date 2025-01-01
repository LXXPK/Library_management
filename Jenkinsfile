pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning Repository'
                git branch: 'main', url: 'https://github.com/LXXPK/Library_management.git'
            }
        }
        stage('Install PHP') {
            steps {
                echo 'Checking PHP Installation'
                bat '''
                REM Check if PHP is installed and install manually if not found
                php -v >nul 2>&1
                if %ERRORLEVEL% NEQ 0 (
                    echo PHP not found, downloading and installing PHP...
                    REM Provide steps to manually install PHP, for example by downloading PHP
                    REM After installing, set it in the environment variables (as done earlier)
                ) else (
                    echo PHP is already installed.
                )
                '''
            }
        }
        stage('Enable mysqli Extension') {
            steps {
                echo 'Enabling mysqli Extension'
                bat '''
                REM Check and enable mysqli extension in php.ini if not enabled
                php -r "if (!extension_loaded('mysqli')) exit(1);"
                if %ERRORLEVEL% neq 0 (
                    echo Enabling mysqli extension...
                    for %%F in ("C:\php\php.ini") do (
                        findstr /C:"extension=mysqli" "%%F" > nul || (
                            echo extension=mysqli >> "%%F"
                        )
                    )
                )
                '''
            }
        }
        stage('Run Tests') {
            steps {
                echo 'Running Tests'
                bat '''
                REM Perform basic PHP syntax check
                php -l index.php
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image'
                bat '''
                REM Ensure Docker is installed and running
                docker --version
                REM Build the Docker image
                docker build -t library-management:latest .
                '''
            }
        }
        stage('Push to Docker Hub') {
            environment {
                DOCKER_CREDENTIALS = credentials('docker-hub-credentials') // Jenkins credentials ID
            }
            steps {
                echo 'Pushing Docker Image to Docker Hub'
                bat '''
                REM Log in to Docker Hub using credentials
                docker login -u %DOCKER_CREDENTIALS_USR% -p %DOCKER_CREDENTIALS_PSW%
                REM Tag and push the Docker image
                docker tag library-management:latest %DOCKER_CREDENTIALS_USR%/library-management:latest
                docker push %DOCKER_CREDENTIALS_USR%/library-management:latest
                '''
            }
        }
        stage('Deploy Application') {
            steps {
                echo 'Deploying Application'
                bat '''
                REM Deploy application to IIS web root
                xcopy /E /I /Y .\\* "C:\\inetpub\\wwwroot\\LibraryManagementSystem\\"
                '''
            }
        }
    }
    post {
        success {
            echo 'Build, Containerization, and Deployment Successful!'
        }
        failure {
            echo 'Build or Deployment failed. Check logs.'
        }
    }
}
