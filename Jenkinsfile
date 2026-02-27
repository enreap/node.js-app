pipeline {
    agent any
    
    tools {
        nodejs "nodejs"   
	}

    environment {
        // Repository & Branch
        GIT_REPO_URL = 'https://github.com/enreap/node.js-app.git'
        BRANCH_NAME  = 'main'

        // Tool names as configured in Jenkins
        SONARQUBE_ENV = 'sonar-scanner'  // Manage Jenkins → Configure System → SonarQube servers
        MAVEN_HOME    = tool 'maven'     // Global Tool Configuration
        JAVA_HOME     = tool 'jdk-21'    // Global Tool Configuration → JDK
        PATH          = "${JAVA_HOME}/bin:${PATH}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning code from ${GIT_REPO_URL} (branch: ${BRANCH_NAME})"
                git branch: "${BRANCH_NAME}", url: "${GIT_REPO_URL}"
            }
        }
        stage('Check Node & NPM Version') {
            steps {
                sh 'node -v'
                sh 'npm -v'
            }        
        }        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }      
		
        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube static code analysis..."
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    withCredentials([string(credentialsId: 'sonartoken1', variable: 'SONAR_TOKEN')]) {
					    sh """
                            /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonar-scanner/bin/sonar-scanner \
                              -Dsonar.projectKey=node.js-app \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=http://3.216.226.173:9000 \
							  -Dsonar.token=sqp_56d19060dd79f885921aedf14d4739d4ab02ef63 \\ 
                        """

                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    echo "Waiting for SonarQube Quality Gate..."
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
    }
}


from job

pipeline {
    agent any

    environment {

        // Git repository & branch
        GIT_REPO_URL = 'https://github.com/enreap/node.js-app.git'
        BRANCH_NAME  = 'main'

        // Sonar Scanner tool name from Jenkins
        SONARQUBE_ENV = 'sonar-scanner'

        // SonarQube project info
        SONAR_PROJECT_KEY = "node.js-app"
        SONAR_HOST_URL    = "http://3.216.226.173:9000"
        DOCKER_IMAGE = "enreapdevopsteam/sonar-project-demo"
        DOCKER_USERNAME = "enreapdevopsteam"
        DOCKER_PAT = "dckr_pat_rXL6w8JnJahrGp06JlL1TVPW-gk"  // Hardcoded (not secure)
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Cloning ${GIT_REPO_URL} (branch: ${BRANCH_NAME})"
                git branch: "${BRANCH_NAME}", url: "${GIT_REPO_URL}"
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Installing npm dependencies..."
                sh "npm install"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube Analysis..."

                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    withCredentials([string(credentialsId: 'sonartoken1', variable: 'SONAR_TOKEN')]) {

                        script {
                            def scannerHome = tool SONARQUBE_ENV

                            sh """
                                ${scannerHome}/bin/sonar-scanner \\
                                  -Dsonar.projectKey=node.js-app \\
                                  -Dsonar.sources=. \\
                                  -Dsonar.host.url=http://3.216.226.173:9000 \\
                                  -Dsonar.token=sqp_56d19060dd79f885921aedf14d4739d4ab02ef63
                            """
                        }
                    }
                }
            }
        }

       //stage('Quality Gate') {
         //   steps {
           //     script {
             //       echo "Waiting for Quality Gate status..."
               //     timeout(time: 5, unit: 'MINUTES') {
                 //       waitForQualityGate abortPipeline: true
                  //  }
                //}
            //}
        //}

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                    echo $DOCKER_PAT | docker login -u $DOCKER_USERNAME --password-stdin
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                """
            }
        }        
    }


    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed — Check Jenkins logs."
        }
    }
}
