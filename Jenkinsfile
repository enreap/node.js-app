pipeline {
    agent any

    tools {
        nodejs 'nodejs'
    }

    environment {
        GIT_REPO_URL = 'https://github.com/enreap/node.js-app.git'
        BRANCH_NAME  = 'main'

        SONAR_SERVER = 'sonarqube'
        SONAR_SCANNER = 'sonar-scanner'

        DOCKER_IMAGE = "enreapdevopsteam/nodejs-project-demo"
        DOCKER_USERNAME = "enreapdevopsteam"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${BRANCH_NAME}", url: "${GIT_REPO_URL}"
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {

                    withCredentials([string(credentialsId: 'sonarqubetoken', variable: 'SONAR_TOKEN')]) {

                        script {
                            def scannerHome = tool "${SONAR_SCANNER}"

                            sh """
                                ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=node.js-app \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://3.216.226.173:9000 \
                                -Dsonar.token=$SONAR_TOKEN
                            """
                        }
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-pat',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                  sh """
                      echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                      docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                  """
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
