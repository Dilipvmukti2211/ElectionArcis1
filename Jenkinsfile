pipeline {
    agent any

    environment {
        DEPLOY_USER = 'deploy'
        DEPLOY_HOST = 'fail.vmukti.com'
        FRONTEND_DIR = '/home/vmukti/electionfrontend'
        BACKEND_DIR = '/home/vmukti/electionbackend'
        GIT_REPO = 'https://github.com/Dilipvmukti2211/ElectionArcis1.git'
        PROJECT_DIR = 'ElectionArcis'
        NODE_ENV = 'production'
        PORT = '3000'
    }

    stages {
        stage('Clone Project') {
            steps {
                echo 'Cloning project...'
                sh """
                    rm -rf $PROJECT_DIR
                    git clone $GIT_REPO $PROJECT_DIR
                """
            }
        }

        stage('Build Frontend') {
            steps {
                echo 'Building frontend...'
                sh """
                    cd $PROJECT_DIR/arcis_frontend_R-D
                    npm install
                    CI=false npm run build
                    cd ../../
                """
            }
        }

        stage('Install Backend') {
            steps {
                echo 'Installing backend...'
                sh """
                    cd $PROJECT_DIR/arcis_backend_R-D
                    npm install
                    cd ../../
                """
            }
        }

        stage('Deploy Frontend') {
            steps {
                echo 'Deploying frontend to server...'
                sh """
                    ssh $DEPLOY_USER@$DEPLOY_HOST \\
                    "sudo mkdir -p $FRONTEND_DIR && sudo chown -R $DEPLOY_USER:$DEPLOY_USER $FRONTEND_DIR && rm -rf $FRONTEND_DIR/*"
                    scp -r $PROJECT_DIR/arcis_frontend_R-D/build/* $DEPLOY_USER@$DEPLOY_HOST:$FRONTEND_DIR/
                """
            }
        }

        stage('Install and Configure Nginx') {
            steps {
                echo 'Installing and configuring Nginx...'
                sh """
                    ssh $DEPLOY_USER@$DEPLOY_HOST \\
                    'if ! command -v nginx > /dev/null; then
                        sudo apt update
                        sudo apt install nginx -y
                    fi

                    FRONTEND_CONF=/etc/nginx/sites-available/electionarcis
                    sudo bash -c "cat > \$FRONTEND_CONF" << EOL
server {
    listen 80;
    server_name _;

    root $FRONTEND_DIR;
    index index.html index.htm;

    location / {
        try_files \$uri /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:$PORT/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_cache_bypass \$http_upgrade;
    }
}
EOL
                    sudo ln -sf /etc/nginx/sites-available/electionarcis /etc/nginx/sites-enabled/
                    sudo nginx -t
                    sudo systemctl restart nginx
                    '
                """
            }
        }

        stage('Deploy Backend') {
            steps {
                echo 'Deploying backend to server...'
                sh """
                    ssh $DEPLOY_USER@$DEPLOY_HOST \\
                    "sudo mkdir -p $BACKEND_DIR && sudo chown -R $DEPLOY_USER:$DEPLOY_USER $BACKEND_DIR && rm -rf $BACKEND_DIR/*"
                    scp -r $PROJECT_DIR/arcis_backend_R-D/* $DEPLOY_USER@$DEPLOY_HOST:$BACKEND_DIR/
                """
            }
        }

        stage('Restart Backend Service') {
            steps {
                echo 'Restarting backend service'
                sh """
                    ssh $DEPLOY_USER@$DEPLOY_HOST \\
                    'SERVICE_FILE=/etc/systemd/system/electionarcis.service

                    if [ ! -f \$SERVICE_FILE ]; then
                        sudo bash -c "cat > \$SERVICE_FILE" << EOL
[Unit]
Description=ElectionArcis Backend Service
After=network.target

[Service]
User=$DEPLOY_USER
WorkingDirectory=$BACKEND_DIR
ExecStart=$(which node) server.js
Restart=always
Environment=NODE_ENV=$NODE_ENV
Environment=PORT=$PORT

[Install]
WantedBy=multi-user.target
EOL
                        sudo systemctl daemon-reload
                        sudo systemctl enable electionarcis
                    fi

                    sudo systemctl restart electionarcis
                    sudo systemctl status electionarcis --no-pager
                    '
                """
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
