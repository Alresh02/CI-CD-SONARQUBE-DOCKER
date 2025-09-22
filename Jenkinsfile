pipeline {
  agent any

  environment {
    SONAR_TOKEN = credentials('sonarqube-token')       // secret text
    DOCKERHUB_REPO = "reshars/ci-cd-sonar-docker-python" 
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-token') // username/password type
    DOCKER_IMAGE = "reshars/ci-cd-sonar-docker-python"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm   // safer for multibranch jobs; or keep explicit git if needed
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

    stage('SonarQube Analysis') {
      steps {
        script {
          // name must match the SonarQube server entry in Jenkins global config
          def scannerHome = tool 'SonarQubeScanner' 

          withSonarQubeEnv('mysonarqube') { 
            bat """
              call .venv\\Scripts\\activate.bat
              REM exclude venv/tests to avoid noise
              "${scannerHome}\\bin\\sonar-scanner.bat" ^
                -Dsonar.projectKey=Alresh02_CI_CD_SONARQUBE_DOCKER ^
                -Dsonar.sources=. ^
                -Dsonar.host.url=http://localhost:9000 ^
                -Dsonar.login=%SONAR_TOKEN% ^
                -Dsonar.exclusions=**/.venv/**,**/venv/**,**/tests/**,**/__pycache__/**,**/*.pyc
            """
          }
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
    stage('Pull Docker Image') {
      steps {
        bat "docker pull %DOCKERHUB_REPO%:latest"
      }
    }

    stage('Deploy Container') {
      steps {
        bat "docker run -d -p 8000:8000 ${env.DOCKER_IMAGE}:latest"
      }
    }
  }
}
