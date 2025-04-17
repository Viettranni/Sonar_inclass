pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQubeServer'
        SONAR_TOKEN = credentials('sonar-token') // store this securely in Jenkins Credentials
        DOCKER_IMAGE = 'viettranni/sonarqube2' // change to your Docker Hub repo
        DOCKER_TAG = 'latest_v1'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Viettranni/Sonar_inclass.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh '''
                        export PATH=$PATH:/Applications/sonar-scanner-7.0.2.4839-macosx-aarch64/bin
                        sonar-scanner \
                          -Dsonar.projectKey=myproject \
                          -Dsonar.sources=src \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Set up Docker Buildx') {
            steps {
                script {
                    sh 'docker buildx create --use || true'
                    sh 'docker buildx inspect --bootstrap'
                }
            }
        }

        stage('Build Multi-Platform Docker Image') {
            steps {
                script {
                    sh """
                        docker buildx build --platform linux/amd64,linux/arm64 -t ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} --push .
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'viettranni', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
                    """
                }
            }
        }
    }
}
