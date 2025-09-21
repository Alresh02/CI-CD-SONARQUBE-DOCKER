pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonarcloud-token')   // Jenkins secret ID
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-token')
        DOCKER_IMAGE = "reshars/ci-cd-sonar-docker-python"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Alresh02/CI-CD-SONARQUBE-DOCKER.git'
            }
        }

        stage('Install dependencies & Run Tests') {
            steps {
                bat '''
                    python -m venv .venv
                    call .venv\\Scripts\\activate.bat
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                    python -m pytest --maxfail=1 --disable-warnings -q
                '''
            }
        }
}

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    bat '''
                    call .venv\\Scripts\\activate.bat
                    python -m pip install sonar-scanner    REM optional if you want python wrapper; otherwise use sonar-scanner.bat or docker scanner
                    sonar-scanner.bat -Dsonar.projectKey=Alresh02_CI-CD-SONARQUBE-DOCKER -Dsonar.organization=alresh02 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=%SONAR_TOKEN%
                    '''
                }
            }
        }
    }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
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
