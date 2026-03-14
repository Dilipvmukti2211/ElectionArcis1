pipeline {
agent any

```
environment {
    DEPLOY_USER = "deploy"
    DEPLOY_HOST = "fail.vmukti.com"
    SSH_KEY = "/var/lib/jenkins/.ssh/jenkins_deploy"

    FRONTEND_DIR = "/var/www/html"
    BACKEND_DIR = "/var/www/electionarcis"
}

stages {

    stage('Checkout') {
        steps {
            git branch: 'main', url: 'https://github.com/Dilipvmukti2211/ElectionArcis1.git'
        }
    }

    stage('Build Frontend') {
        steps {
            sh '''
            cd arcis_frontend_R-D
            npm install
            CI=false npm run build
            '''
        }
    }

    stage('Install Backend') {
        steps {
            sh '''
            cd arcis_backend_R-D
            npm install
            '''
        }
    }

    stage('Deploy Frontend') {
        steps {
            sh '''
            ssh -o StrictHostKeyChecking=no -i $SSH_KEY $DEPLOY_USER@$DEPLOY_HOST "sudo rm -rf $FRONTEND_DIR/*"
            scp -o StrictHostKeyChecking=no -i $SSH_KEY -r arcis_frontend_R-D/build/* $DEPLOY_USER@$DEPLOY_HOST:$FRONTEND_DIR/
            '''
        }
    }

    stage('Deploy Backend') {
        steps {
            sh '''
            ssh -o StrictHostKeyChecking=no -i $SSH_KEY $DEPLOY_USER@$DEPLOY_HOST "sudo rm -rf $BACKEND_DIR/*"
            scp -o StrictHostKeyChecking=no -i $SSH_KEY -r arcis_backend_R-D/* $DEPLOY_USER@$DEPLOY_HOST:$BACKEND_DIR/
            '''
        }
    }

    stage('Restart Backend') {
        steps {
            sh '''
            ssh -o StrictHostKeyChecking=no -i $SSH_KEY $DEPLOY_USER@$DEPLOY_HOST "sudo systemctl restart electionarcis"
            '''
        }
    }

}

post {
    success {
        echo "Deployment Successful"
    }
    failure {
        echo "Deployment Failed"
    }
}
```

}
