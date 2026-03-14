stage('Test SSH') {
steps {
sh '''
ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/jenkins_deploy [deploy@fail.vmukti.com](mailto:deploy@fail.vmukti.com) "echo SSH CONNECTION SUCCESS"
'''
}
}
