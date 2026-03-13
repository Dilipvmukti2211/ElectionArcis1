#!/bin/bash
set -e

# -----------------------
# Config
# -----------------------
PROJECT_DIR=/home/vmukti/ElectionArcis
FRONTEND_DIR=/home/vmukti/electionfrontend
BACKEND_DIR=/home/vmukti/electionbackend
GIT_REPO=https://github.com/Dilipvmukti2211/ElectionArcis1.git
NODE_ENV=production
PORT=3000

# -----------------------
# Clone or Pull Project
# -----------------------
cd /home/vmukti
if [ -d "$PROJECT_DIR" ]; then
    echo "Updating existing project..."
    cd "$PROJECT_DIR"
    git reset --hard
    git pull
else
    echo "Cloning project..."
    git clone "$GIT_REPO" "$PROJECT_DIR"
fi

# -----------------------
# Build Frontend
# -----------------------
echo "Building frontend..."
cd "$PROJECT_DIR/arcis_frontend_R-D"
npm install
CI=false npm run build

# Prepare frontend folder
sudo mkdir -p "$FRONTEND_DIR"
sudo rm -rf "$FRONTEND_DIR/*"
sudo cp -r build/* "$FRONTEND_DIR"
sudo chown -R vmukti:vmukti "$FRONTEND_DIR"

# -----------------------
# Install Backend
# -----------------------
echo "Installing backend..."
cd "$PROJECT_DIR/arcis_backend_R-D"
npm install

# Deploy backend
sudo mkdir -p "$BACKEND_DIR"
sudo rm -rf "$BACKEND_DIR/*"
sudo cp -r * "$BACKEND_DIR"
sudo chown -R vmukti:vmukti "$BACKEND_DIR"

# -----------------------
# Setup Backend Service
# -----------------------
SERVICE_FILE=/etc/systemd/system/electionarcis.service
if [ ! -f "$SERVICE_FILE" ]; then
sudo bash -c "cat > $SERVICE_FILE" <<EOL
[Unit]
Description=ElectionArcis Backend Service
After=network.target

[Service]
User=vmukti
WorkingDirectory=$BACKEND_DIR
ExecStart=/usr/bin/env node server.js
Restart=always
Environment=NODE_ENV=$NODE_ENV
Environment=PORT=$PORT

[Install]
WantedBy=multi-user.target
EOL
    sudo systemctl daemon-reload
    sudo systemctl enable electionarcis
fi

# Restart backend service
sudo systemctl restart electionarcis
sudo systemctl status electionarcis --no-pager

# -----------------------
# Configure Nginx
# -----------------------
FRONTEND_CONF=/etc/nginx/sites-available/electionarcis
sudo bash -c "cat > $FRONTEND_CONF" <<EOL
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

echo "Deployment completed successfully!"
