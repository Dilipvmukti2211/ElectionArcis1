pipeline {
    agent any

    stages {
        stage('Test SSH') {
            steps {
                echo 'Testing SSH connection...'

                withCredentials([sshUserPrivateKey(
                    credentialsId: 'deploy-ssh-key',
                    keyFileVariable: 'SSH_KEY',
                    usernameVariable: 'SSH_USER'
                )]) {

                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY $SSH_USER@fail.vmukti.com "echo SSH SUCCESS"
                    '''
                }
            }
        }
    }
}
