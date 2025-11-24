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
                            ${MAVEN_HOME}/bin/mvn clean verify sonar:sonar \\
								-Dsonar.projectKey=node.js-app \\
								-Dsonar.projectName='node.js-app' \\
								-Dsonar.host.url=http://3.216.226.173:9000 \\
								-Dsonar.token=sqp_56d19060dd79f885921aedf14d4739d4ab02ef63 \\
								-Dsonar.qualitygate.wait=true
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



// pipeline {
//     agent any

//     environment {
//         // SonarQube token stored as Jenkins secret text
//         SONAR_TOKEN = credentials('sonar-token-id')

//         // Nexus credentials stored in Jenkins (username/password)
//         NEXUS_USER = credentials('nexus-username')
//         NEXUS_PASSWORD = credentials('nexus-password')

//         // Node.js version configured in Jenkins Global Tool Configuration
//         NODEJS_HOME = tool name: 'nodejs22.40.0', type: 'NodeJS'
//         PATH = "${tool 'nodejs22.40.0'}/bin:${env.PATH}"
//     }

//     stages {

//         stage('Checkout Code') {
//             steps {
//                 git(
//                     url: 'https://github.com/enreap/node.js-app.git',
//                     credentialsId: 'f52a7301-4cbc-4389-91fd-0e6ef69c493d'
//                 )
//             }
//         }

//         stage('Install Dependencies') {
//             steps {
//                 sh 'npm ci' // clean CI install
//             }
//         }

//         stage('Run Tests') {
//             steps {
//                 sh 'npm test'
//             }
//         }

//         stage('SonarQube Analysis') {
//             steps {
//                 withEnv(["SONAR_TOKEN=${SONAR_TOKEN}"]) {
//                     sh 'npm run sonar' // npm script should reference ${SONAR_TOKEN} for login
//                 }
//             }
//         }

//         stage('Publish to Nexus (Optional)') {
//             when {
//                 branch 'main' // only publish from main branch
//             }
//             steps {
//                 withCredentials([usernamePassword(credentialsId: 'nexus-credentials-id', 
//                                                   passwordVariable: 'NEXUS_PASSWORD', 
//                                                   usernameVariable: 'NEXUS_USER')]) {
//                     sh '''
//                         npm set registry http://18.130.99.18:9980/mithuntechnologies/repository/nodejsmithuntechnologies/
//                         npm login --registry=http://18.130.99.18:9980/mithuntechnologies/repository/nodejsmithuntechnologies/ --username $NEXUS_USER --password $NEXUS_PASSWORD
//                         npm publish
//                     '''
//                 }
//             }
//         }

//         stage('Run Node.js App') {
//             steps {
//                 sh 'npm start'
//             }
//         }

//     }

//     post {
//         always {
//             echo 'Cleaning workspace'
//             cleanWs()
//         }
//         success {
//             echo 'Pipeline completed successfully!'
//         }
//         failure {
//             echo 'Pipeline failed. Check logs for details.'
//         }
//     }
// }


// pipeline {
//     agent any

//     environment {
//         // SonarQube token stored as Jenkins secret text
//         SONAR_TOKEN = credentials('sonar-token-id')

//         // Node.js version configured in Jenkins Global Tool Configuration
//         NODEJS_HOME = tool name: 'nodejs22.40.0', type: 'NodeJS'
//         PATH = "${tool 'nodejs22.40.0'}/bin:${env.PATH}"
//     }

//     stages {

//         stage('Checkout Code') {
//             steps {
//                 git(
//                     url: 'https://github.com/enreap/node.js-app.git',
//                     credentialsId: 'f52a7301-4cbc-4389-91fd-0e6ef69c493d'
//                 )
//             }
//         }

//         stage('Install Dependencies') {
//             steps {
//                 sh 'npm ci' // clean install for CI
//             }
//         }

//         stage('SonarQube Analysis') {
//             steps {
//                 withEnv(["SONAR_TOKEN=${SONAR_TOKEN}"]) {
//                     sh 'npm run sonar' // npm script should use SONAR_TOKEN for authentication
//                 }
//             }
//         }

//     }

//     post {
//         always {
//             echo 'Cleaning workspace'
//             cleanWs()
//         }
//         success {
//             echo 'Node.js build and SonarQube scan completed successfully!'
//         }
//         failure {
//             echo 'Pipeline failed. Check logs for details.'
//         }
//     }
// }
