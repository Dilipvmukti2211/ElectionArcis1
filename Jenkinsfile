pipeline {
    agent any

    environment {
        DEPLOY_USER = "deploy"
        DEPLOY_HOST = "fail.vmukti.com"
        FRONTEND_DIR = "/home/deploy/electionarcis/frontend"
        BACKEND_DIR = "/home/deploy/electionarcis/backend"
    }

    stages {

        stage('Clone Project') {
            steps {
                echo "Cloning repository..."
                git branch: 'main', url: 'https://github.com/Dilipvmukti2211/ElectionArcis1.git'
            }
        }

        stage('Test SSH Connection') {
            steps {

                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {

                    sh '''
                    chmod 600 $SSH_KEY
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "echo SSH CONNECTED"
                    '''
                }
            }
        }

        stage('Install PM2 (if not installed)') {
            steps {

                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {

                    sh '''
                    ssh -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "
                        if ! command -v pm2 &> /dev/null
                        then
                            echo 'Installing PM2...'
                            sudo npm install -g pm2
                        else
                            echo 'PM2 already installed'
                        fi
                    "
                    '''
                }
            }
        }

        stage('Deploy Frontend') {
            steps {

                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {

                    sh '''
                    chmod 600 $SSH_KEY

                    ssh -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "mkdir -p ${FRONTEND_DIR}"

                    scp -i $SSH_KEY -r arcis_frontend_R-D/* \
                    ${DEPLOY_USER}@${DEPLOY_HOST}:${FRONTEND_DIR}

                    ssh -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "
                        cd ${FRONTEND_DIR}
                        npm install
                        pm2 restart frontend || pm2 start npm --name frontend -- start
                    "
                    '''
                }
            }
        }

        stage('Deploy Backend') {
            steps {

                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY'
                )]) {

                    sh '''
                    chmod 600 $SSH_KEY

                    ssh -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "mkdir -p ${BACKEND_DIR}"

                    scp -i $SSH_KEY -r arcis_backend_R-D/* \
                    ${DEPLOY_USER}@${DEPLOY_HOST}:${BACKEND_DIR}

                    ssh -i $SSH_KEY ${DEPLOY_USER}@${DEPLOY_HOST} "
                        cd ${BACKEND_DIR}
                        npm install
                        pm2 restart backend || pm2 start server.js --name backend
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
