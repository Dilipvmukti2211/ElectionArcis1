pipeline {
agent any

```
environment {
    DEPLOY_USER  = "deploy"
    DEPLOY_HOST  = "fail.vmukti.com"
    FRONTEND_DIR = "/var/www/html"
    BACKEND_DIR  = "/var/www/electionarcis"
    SSH_KEY      = "/var/lib/jenkins/.ssh/jenkins_deploy"
    NODE_ENV     = "production"
    PORT         = "3000"
}

stages {

    stage('Checkout Code') {
        steps {
            echo "Cloning repository..."
            git branch: 'main', url: 'https://github.com/Dilipvmukti2211/ElectionArcis1.git'
        }
    }

    stage('Build Frontend') {
        steps {
            echo "Building React frontend..."
            sh '''
                cd arcis_frontend_R-D
                npm install
                CI=false npm run build
            '''
        }
    }

    stage('Install Backend Dependencies') {
        steps {
            echo "Installing backend packages..."
            sh '''
                cd arcis_backend_R-D
                npm install
            '''
        }
    }

    stage('Deploy Frontend') {
        steps {
            echo "Deploying frontend build..."
            sh '''
                ssh -o StrictHostKeyChecking=no -i $SSH_KEY $DEPLOY_USER@$DEPLOY_HOST \
                "sudo mkdir -p $FRONTEND_DIR && sudo rm -rf $FRONTEND_DIR/*"

                scp -o StrictHostKeyChecking=no -i $SSH_KEY -r \
                arcis_frontend_R-D/build/* \
                $DEPLOY_USER@$DEPLOY_HOST:$FRONTEND_DIR/
            '''
        }
    }

    stage('Configure Nginx') {
        steps {
            echo "Configuring Nginx..."
            sh '''
```

ssh -o StrictHostKeyChecking=no -i $SSH_KEY $DEPLOY_USER@$DEPLOY_HOST << 'ENDSSH'

if ! command -v nginx > /dev/null; then
sudo apt update -y
sudo apt install nginx -y
fi

sudo tee /etc/nginx/sites-available/electionarcis > /dev/null << 'NGINXCONF'
server {
listen 80;
server_name _;

```
root /var/www/html;
index index.html;

location / {
    try_files $uri /index.html;
}

location /api/ {
    proxy_pass http://127.0.0.1:3000/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
}
```

}
NGINXCONF

sudo ln -sf /etc/nginx/sites-available/electionarcis /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx

ENDSSH
'''
}
}

```
    stage('Deploy Backend') {
        steps {
            echo "Deploying backend files..."
            sh '''
                ssh -o StrictHostKeyChecking=no -i $SSH_KEY $DEPLOY_USER@$DEPLOY_HOST \
                "sudo mkdir -p $BACKEND_DIR && sudo rm -rf $BACKEND_DIR/*"

                scp -o StrictHostKeyChecking=no -i $SSH_KEY -r \
                arcis_backend_R-D/* \
                $DEPLOY_USER@$DEPLOY_HOST:$BACKEND_DIR/
            '''
        }
    }

    stage('Restart Backend Service') {
        steps {
            echo "Restarting Node backend service..."
            sh '''
```

ssh -o StrictHostKeyChecking=no -i $SSH_KEY $DEPLOY_USER@$DEPLOY_HOST << 'ENDSSH'

SERVICE_FILE=/etc/systemd/system/electionarcis.service

if [ ! -f $SERVICE_FILE ]; then
sudo tee $SERVICE_FILE > /dev/null << 'SERVICECONF'
[Unit]
Description=ElectionArcis Backend
After=network.target

[Service]
User=deploy
WorkingDirectory=/var/www/electionarcis
ExecStart=/usr/bin/node /var/www/electionarcis/server.js
Restart=always
Environment=NODE_ENV=production
Environment=PORT=3000

[Install]
WantedBy=multi-user.target
SERVICECONF

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

```
post {
    success {
        echo "Deployment completed successfully"
    }
    failure {
        echo "Deployment FAILED. Check Jenkins logs."
    }
}
```

}
