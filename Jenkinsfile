pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonarcloud-token')   // Jenkins credential ID
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds-token')
        DOCKER_IMAGE = "alresh02/ci-cd-sonar-docker-python"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Alresh02/CI-CD-SONARQUBE-DOCKER.git'
            }
        }

        stage('Install dependencies & Run Tests') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pytest --maxfail=1 --disable-warnings -q'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=Alresh02_CI-CD-SONARQUBE-DOCKER \
                          -Dsonar.organization=alresh02 \
                          -Dsonar.host.url=https://sonarcloud.io \
                          -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        def app = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                        app.push()
                        app.push("latest")
                    }
                }
            }
        }

        stage('Deploy (Run Locally)') {
            steps {
                sh "docker run -d -p 8000:8000 ${DOCKER_IMAGE}:latest"
            }
        }
    }
}
