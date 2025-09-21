pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonarcloud-token')   // Jenkins secret ID (secret text)
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-token') // username/password credentials ID
        DOCKER_IMAGE = "reshars/ci-cd-sonar-docker-python"
    }

    stages {
        stage('Checkout') {
            steps {
                // safer: use checkout scm if job is multibranch; your explicit git is fine too
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

        stage('SonarCloud Analysis') {
            steps {
                script {
                      // exact name from Global Tool Configuration
                      def scannerHome = tool 'SonarQubeScanner' 
                withSonarQubeEnv('SonarCloud') {
                    bat """
                      call .venv\\Scripts\\activate.bat
                      "${scannerHome}\\bin\\sonar-scanner.bat" -Dsonar.projectKey=Alresh02_CI-CD-SONARQUBE-DOCKER -Dsonar.organization=alresh02 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=%SONAR_TOKEN%
                    """
                  }
                }
            }
        }


        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // requires SonarQube plugin and that the analysis completed (report-task.txt)
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def tag = "${env.BUILD_NUMBER}"
                    bat """
                        docker build -t ${env.DOCKER_IMAGE}:${tag} .
                        docker login -u %DOCKERHUB_CREDENTIALS_USR% -p %DOCKERHUB_CREDENTIALS_PSW%
                        docker tag ${env.DOCKER_IMAGE}:${tag} ${env.DOCKER_IMAGE}:latest
                        docker push ${env.DOCKER_IMAGE}:${tag}
                        docker push ${env.DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy Container') {
            steps {
                bat "docker run -d -p 8000:8000 ${env.DOCKER_IMAGE}:latest"
            }
        }
    }
}
