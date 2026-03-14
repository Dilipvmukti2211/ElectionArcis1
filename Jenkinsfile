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

                sshagent(['deploy-ssh-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST "echo SSH SUCCESS && hostname && whoami"
                    '''
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                echo "Deploying frontend..."

                sshagent(['deploy-ssh-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST "sudo mkdir -p $FRONTEND_DIR && sudo rm -rf $FRONTEND_DIR/*"

                    scp -o StrictHostKeyChecking=no -r arcis_frontend_R-D/build/* $DEPLOY_USER@$DEPLOY_HOST:$FRONTEND_DIR/
                    '''
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                echo "Deploying backend..."

                sshagent(['deploy-ssh-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST "sudo mkdir -p $BACKEND_DIR && sudo rm -rf $BACKEND_DIR/*"

                    scp -o StrictHostKeyChecking=no -r arcis_backend_R-D/* $DEPLOY_USER@$DEPLOY_HOST:$BACKEND_DIR/
                    '''
                }
            }
        }

        stage('Restart Backend Service') {
            steps {
                echo "Restarting backend service..."

                sshagent(['deploy-ssh-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST "

                    if ! command -v node > /dev/null; then
                        sudo apt update
                        sudo apt install -y nodejs npm
                    fi

                    if ! command -v pm2 > /dev/null; then
                        sudo npm install -g pm2
                    fi

                    cd $BACKEND_DIR
                    pm2 delete electionarcis || true
                    pm2 start server.js --name electionarcis
                    pm2 save
                    "
                    '''
                }
            }
        }

    }

    post {
        success {
            echo "Deployment completed successfully"
        }

        failure {
            echo "Deployment FAILED"
        }
    }
}
