
pipeline {
    agent any

    environment {
        IMAGE_TAG = 'v1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat '''
                    docker run --rm -v "%CD%:/opt" -w /opt node:7.8.0 npm install
                    docker run --rm -v "%CD%:/opt" -w /opt node:7.8.0 npm run build
                '''
            }
        }

        stage('Test') {
            steps {
                bat '''
                    docker run --rm -e CI=true -v "%CD%:/opt" -w /opt node:7.8.0 npm test -- --watch=false
                '''
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    def branch = env.BRANCH_NAME

                    if (branch == 'main') {
                        env.APP_PORT = '3000'
                        env.IMAGE_NAME = 'nodemain'
                    } else if (branch == 'dev') {
                        env.APP_PORT = '3001'
                        env.IMAGE_NAME = 'nodedev'
                    } else {
                        error("Unsupported branch: ${branch}")
                    }
                }

                bat 'docker build -t %IMAGE_NAME%:%IMAGE_TAG% .'
            }
        }

        stage('Deploy') {
            steps {
                bat '''
                    docker rm -f %IMAGE_NAME% 2>NUL
                    docker run -d --name %IMAGE_NAME% -p %APP_PORT%:3000 -e PORT=3000 %IMAGE_NAME%:%IMAGE_TAG%
                '''
            }
        }
    }

    post {
        success {
            echo 'Build and deployment completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
