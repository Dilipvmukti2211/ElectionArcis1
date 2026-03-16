pipeline {
    agent {
        label 'production'
    }

    environment {
        FRONTEND_DIR = "/home/vmukti/electionarcis/frontend"
        BACKEND_DIR = "/home/vmukti/electionarcis/backend"
        REPO_URL = "https://github.com/Dilipvmukti2211/ElectionArcis1.git"
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo "Cloning latest code..."
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Create App Directories') {
            steps {
                sh '''
                mkdir -p ${FRONTEND_DIR}
                mkdir -p ${BACKEND_DIR}
                '''
            }
        }

        stage('Deploy Backend Code') {
            steps {
                sh '''
                rm -rf ${BACKEND_DIR}/*
                cp -r arcis_backend_R-D/* ${BACKEND_DIR}/
                '''
            }
        }

        stage('Deploy Frontend Code') {
            steps {
                sh '''
                rm -rf ${FRONTEND_DIR}/*
                cp -r arcis_frontend_R-D/* ${FRONTEND_DIR}/
                '''
            }
        }

        stage('Create Frontend ENV') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh '''
                    cat <<EOF > .env
REACT_APP_URL=http://98.70.50.122:8082
REACT_APP_BASE_URL=http://98.70.50.122:8082
REACT_APP_GOOGLE_MAPS_KEY=AIzaSyD2CF3PlGBd0tQhusHwX3ngfPaad0pmJ_Q
REACT_APP_ENCRYPT_KEY=
REACT_APP_IV=
REACT_APP_DECRYPT_KEY=
REACT_APP_D_IV=
EOF
                    '''
                }
            }
        }

        stage('Install Backend Dependencies') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'npm install'
                }
            }
        }

        stage('Install Frontend Dependencies') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh 'npm install'
                }
            }
        }

        stage('Start Backend') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh '''
                    pm2 restart backend || pm2 start server.js --name backend
                    '''
                }
            }
        }

        stage('Start Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh '''
                    pm2 restart frontend || pm2 start npm --name frontend -- start
                    '''
                }
            }
        }

        stage('Save PM2 Processes') {
            steps {
                sh 'pm2 save'
            }
        }

    }

    post {
        success {
            echo "🚀 Deployment Successful"
        }
        failure {
            echo "❌ Deployment Failed"
        }
    }
}
