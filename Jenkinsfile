pipeline {
    agent any

    environment {
        DEPLOY_USER = "deploy"
        DEPLOY_HOST = "fail.vmukti.com"
        FRONTEND_DIR = "/var/www/html"
        BACKEND_DIR = "/var/www/electionarcis"
    }

    stages {

        stage('Clone Project') {
            steps {
                echo "Cloning repository..."
                git branch: 'main', url: 'https://github.com/Dilipvmukti2211/ElectionArcis1.git'
            }
        }

        stage('Create Frontend ENV') {
            steps {
                echo "Creating frontend .env file..."
                sh '''
                cat > arcis_frontend_R-D/.env <<EOF
REACT_APP_URL=http://localhost:8081
REACT_APP_BASE_URL=http://localhost:8081
REACT_APP_GOOGLE_MAPS_KEY=AIzaSyD2CF3PlGBd0tQhusHwX3ngfPaad0pmJ_Q
REACT_APP_ENCRYPT_KEY=
REACT_APP_IV=
REACT_APP_DECRYPT_KEY=
REACT_APP_D_IV=
EOF
                '''
            }
        }

        stage('Build Frontend') {
            steps {
                echo "Building frontend..."
                sh '''
                cd arcis_frontend_R-D
                npm install
                CI=false npm run build
                '''
            }
        }

        stage('Install Backend Dependencies') {
            steps {
                echo "Installing backend dependencies..."
                sh '''
                cd arcis_backend_R-D
                npm install
                '''
            }
        }

        stage('Test SSH Connection') {
            steps {
                echo "Testing SSH connection..."
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    sh '''
                    chmod 600 $SSH_KEY
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "echo SSH SUCCESS && hostname && whoami"
                    '''
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                echo "Deploying frontend..."
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    sh '''
                    chmod 600 $SSH_KEY

                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "
                    sudo mkdir -p ${FRONTEND_DIR}
                    sudo rm -rf ${FRONTEND_DIR}/*
                    "

                    scp -o StrictHostKeyChecking=no -i $SSH_KEY -r arcis_frontend_R-D/build/* ${DEPLOY_USER}@${DEPLOY_HOST}:${FRONTEND_DIR}/
                    '''
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                echo "Deploying backend..."
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    sh '''
                    chmod 600 $SSH_KEY

                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "
                    sudo mkdir -p ${BACKEND_DIR}
                    sudo rm -rf ${BACKEND_DIR}/*
                    "

                    scp -o StrictHostKeyChecking=no -i $SSH_KEY -r arcis_backend_R-D/* ${DEPLOY_USER}@${DEPLOY_HOST}:${BACKEND_DIR}/
                    '''
                }
            }
        }

        stage('Restart Backend Service') {
            steps {
                echo "Restarting backend service..."
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {
                    sh '''
                    chmod 600 $SSH_KEY

                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "
                    sudo systemctl restart electionarcis
                    sudo systemctl status electionarcis --no-pager
                    "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment SUCCESS"
        }
        failure {
            echo "Deployment FAILED"
        }
    }
}
