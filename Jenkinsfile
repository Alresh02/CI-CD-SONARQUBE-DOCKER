pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonarcloud-token')   // Jenkins secret ID
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds-token')
        DOCKER_IMAGE = "reshars/ci-cd-sonar-python"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Alresh02/CI-CD-SONARQUBE-DOCKER.git'
            }
        }

        stage('Install dependencies & Run Tests') {
            steps {
                // Use bat on Windows
                bat '''
                python -m pip install --upgrade pip
                python -m pip install -r requirements.txt
                pytest --maxfail=1 --disable-warnings -q
                '''
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    // If you installed SonarScanner on the Windows agent, run sonar-scanner.bat
                    // Otherwise use the Docker scanner (see notes below).
                    bat """
                    sonar-scanner.bat \
                      -Dsonar.projectKey=Alresh02_CI-CD-SONARQUBE-DOCKER \
                      -Dsonar.organization=alresh02 \
                      -Dsonar.host.url=https://sonarcloud.io \
                      -Dsonar.login=%SONAR_TOKEN%
                    """
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                bat """
                docker build -t ${env.DOCKER_IMAGE}:$BUILD_NUMBER .
                docker login -u %DOCKERHUB_CREDENTIALS_USR% -p %DOCKERHUB_CREDENTIALS_PSW%
                docker tag ${env.DOCKER_IMAGE}:$BUILD_NUMBER ${env.DOCKER_IMAGE}:latest
                docker push ${env.DOCKER_IMAGE}:$BUILD_NUMBER
                docker push ${env.DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Deploy Container') {
            steps {
                bat "docker run -d -p 8000:8000 ${env.DOCKER_IMAGE}:latest"
            }
        }
    }
}
