pipeline {
    agent any

    environment {
        DEPLOY_USER = 'deploy'
        DEPLOY_HOST = 'fail.vmukti.com'
        FRONTEND_DIR = '/var/www/html'
        BACKEND_DIR = '/var/www/electionarcis'
        GIT_REPO = 'https://github.com/Dilipvmukti2211/ElectionArcis1.git'
        PROJECT_DIR = 'ElectionArcis'
        NODE_ENV = 'production'
        PORT = '3000'
    }

    stages {

        stage('Clone Project') {
            steps {
                echo 'Cloning project...'
                sh '''
                    rm -rf $PROJECT_DIR
                    git clone $GIT_REPO $PROJECT_DIR
                '''
            }
        }

        stage('Build Frontend') {
            steps {
                echo 'Building frontend...'
                sh '''
                    cd $PROJECT_DIR/arcis_frontend_R-D
                    npm install
                    CI=false npm run build
                '''
            }
        }

        stage('Install Backend Dependencies') {
            steps {
                echo 'Installing backend...'
                sh '''
                    cd $PROJECT_DIR/arcis_backend_R-D
                    npm install
                '''
            }
        }

        stage('Deploy Frontend') {
            steps {
                echo 'Deploying frontend to server...'
                sshagent(credentials: ['deploy-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST "
                            sudo mkdir -p $FRONTEND_DIR
                            sudo rm -rf $FRONTEND_DIR/*
                        "
                        scp -o StrictHostKeyChecking=no -r \
                            $PROJECT_DIR/arcis_frontend_R-D/build/* \
                            $DEPLOY_USER@$DEPLOY_HOST:$FRONTEND_DIR/
                    '''
                }
            }
        }

        stage('Configure Nginx') {
            steps {
                echo 'Setting up Nginx...'
                sshagent(credentials: ['deploy-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST bash << 'ENDSSH'
if ! command -v nginx > /dev/null; then
    sudo apt update
    sudo apt install nginx -y
fi

FRONTEND_CONF=/etc/nginx/sites-available/electionarcis
sudo bash -c "cat > $FRONTEND_CONF" << 'EOL'
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.html index.htm;

    location / {
        try_files $uri /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
EOL

sudo ln -sf /etc/nginx/sites-available/electionarcis /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
ENDSSH
                    '''
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                echo 'Deploying backend...'
                sshagent(credentials: ['deploy-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST \
                            "sudo mkdir -p $BACKEND_DIR && rm -rf $BACKEND_DIR/*"
                        scp -o StrictHostKeyChecking=no -r \
                            $PROJECT_DIR/arcis_backend_R-D/* \
                            $DEPLOY_USER@$DEPLOY_HOST:$BACKEND_DIR/
                    '''
                }
            }
        }

        stage('Restart Backend Service') {
            steps {
                echo 'Restarting backend via systemd...'
                sshagent(credentials: ['deploy-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST bash << 'ENDSSH'
SERVICE_FILE=/etc/systemd/system/electionarcis.service

if [ ! -f $SERVICE_FILE ]; then
    sudo bash -c "cat > $SERVICE_FILE" << 'EOL'
[Unit]
Description=ElectionArcis Backend Service
After=network.target

[Service]
User=deploy
WorkingDirectory=/var/www/electionarcis
ExecStart=/usr/bin/node server.js
Restart=always
Environment=NODE_ENV=production
Environment=PORT=3000

[Install]
WantedBy=multi-user.target
EOL
    sudo systemctl daemon-reload
    sudo systemctl enable electionarcis
fi

sudo systemctl restart electionarcis
sudo systemctl status electionarcis --no-pager
ENDSSH
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment FAILED. Check logs above.'
        }
    }
}
