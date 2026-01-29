pipeline {
    agent any

    environment {
        SSH_CRED  = 'ssh-jenkinsprod'
        WIN_USER  = 'administrator'
        WIN_HOST  = '192.168.192.131'
        TARGET_DIR = 'C:/inetpub/wwwroot/AKR_LC_FAME'
    }

    stages {

        stage('Deploy New Files (SCP)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
scp -o StrictHostKeyChecking=no -r \
Areas Models Views bin \
${WIN_USER}@${WIN_HOST}:${TARGET_DIR}/
'''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} \
"cmd /c dir \\"C:\\inetpub\\wwwroot\\AKR_LC_FAME\\""
'''
                }
            }
        }
    }

    post {
        success {
            echo '✅ DEPLOY SUCCESS — FILE TERKIRIM DENGAN SELAMAT'
        }
        failure {
            echo '❌ DEPLOY FAILED'
        }
        always {
            cleanWs()
        }
    }
}
