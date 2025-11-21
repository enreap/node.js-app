pipeline {
    agent any

    environment {
        // SonarQube token stored as Jenkins secret text
        SONAR_TOKEN = credentials('sonar-token-id')

        // Nexus credentials stored in Jenkins (username/password)
        NEXUS_USER = credentials('nexus-username')
        NEXUS_PASSWORD = credentials('nexus-password')

        // Node.js version configured in Jenkins Global Tool Configuration
        NODEJS_HOME = tool name: 'nodejs22.40.0', type: 'NodeJS'
        PATH = "${tool 'nodejs22.40.0'}/bin:${env.PATH}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git(
                    url: 'https://github.com/enreap/node.js-app.git',
                    credentialsId: 'f52a7301-4cbc-4389-91fd-0e6ef69c493d'
                )
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci' // clean CI install
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withEnv(["SONAR_TOKEN=${SONAR_TOKEN}"]) {
                    sh 'npm run sonar' // npm script should reference ${SONAR_TOKEN} for login
                }
            }
        }

        stage('Publish to Nexus (Optional)') {
            when {
                branch 'main' // only publish from main branch
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials-id', 
                                                  passwordVariable: 'NEXUS_PASSWORD', 
                                                  usernameVariable: 'NEXUS_USER')]) {
                    sh '''
                        npm set registry http://18.130.99.18:9980/mithuntechnologies/repository/nodejsmithuntechnologies/
                        npm login --registry=http://18.130.99.18:9980/mithuntechnologies/repository/nodejsmithuntechnologies/ --username $NEXUS_USER --password $NEXUS_PASSWORD
                        npm publish
                    '''
                }
            }
        }

        stage('Run Node.js App') {
            steps {
                sh 'npm start'
            }
        }

    }

    post {
        always {
            echo 'Cleaning workspace'
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}


pipeline {
    agent any

    environment {
        // SonarQube token stored as Jenkins secret text
        SONAR_TOKEN = credentials('sonar-token-id')

        // Node.js version configured in Jenkins Global Tool Configuration
        NODEJS_HOME = tool name: 'nodejs22.40.0', type: 'NodeJS'
        PATH = "${tool 'nodejs22.40.0'}/bin:${env.PATH}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git(
                    url: 'https://github.com/enreap/node.js-app.git',
                    credentialsId: 'f52a7301-4cbc-4389-91fd-0e6ef69c493d'
                )
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci' // clean install for CI
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withEnv(["SONAR_TOKEN=${SONAR_TOKEN}"]) {
                    sh 'npm run sonar' // npm script should use SONAR_TOKEN for authentication
                }
            }
        }

    }

    post {
        always {
            echo 'Cleaning workspace'
            cleanWs()
        }
        success {
            echo 'Node.js build and SonarQube scan completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
