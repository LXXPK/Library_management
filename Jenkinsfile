pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning Repository'
                git branch: 'main', url: 'https://github.com/LXXPK/Library_management.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                echo 'Installing PHP and Enabling mysqli Extension'
                bat '''
                REM Install PHP if not installed
                choco install php -y || echo "PHP is already installed"
                
                REM Check and enable mysqli extension
                php -r "if (!extension_loaded('mysqli')) exit(1);"
                if %ERRORLEVEL% neq 0 (
                    echo Enabling mysqli extension...
                    for %%F in ("php.ini", "C:\\tools\\php\\php.ini") do (
                        findstr /C:"extension=mysqli" "%%F" > nul || (
                            echo Updating %%F to enable mysqli...
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
                docker build -t gym-management-system:latest .
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
                docker tag gym-management-system:latest %DOCKER_CREDENTIALS_USR%/gym-management-system:latest
                docker push %DOCKER_CREDENTIALS_USR%/gym-management-system:latest
                '''
            }
        }
        stage('Deploy Application') {
            steps {
                echo 'Deploying Application'
                bat '''
                REM Deploy application to IIS web root
                xcopy /E /I /Y .\\* "C:\\inetpub\\wwwroot\\GymManagementSystem\\"
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
