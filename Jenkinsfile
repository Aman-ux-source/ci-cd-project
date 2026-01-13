pipeline {
    agent any

    environment {
        IMAGE_NAME = "amanuxsource/ci-cd-project"
        IMAGE_TAG  = "latest"
        SONAR_SCANNER = tool 'sonar-scanner' // the name you gave in Jenkins Tools
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Aman-ux-source/ci-cd-project.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') { // the name you gave in Jenkins SonarQube config
                    sh """
                    ${SONAR_SCANNER}/bin/sonar-scanner \
                    -Dsonar.projectKey=ci-cd-project \
                    -Dsonar.projectName=ci-cd-project \
                    -Dsonar.sources=. \
                    -Dsonar.language=web
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker rm -f ci-cd-project || true
                docker run -d -p 80:80 --name ci-cd-project $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
