pipeline {
    agent any

    environment {
        SSH_CRED   = 'ssh-jenkinsprod'
        TARGET_IP = '192.168.192.131'
        APP_PATH  = 'C:/inetpub/wwwroot/AKR_LC_FAME'
        BACKUP_TO = 'E:/BACKUP/AFTER'
    }

    stages {

        stage('Backup Existing Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no administrator@${TARGET_IP} \
                    "powershell -NoProfile -Command \\
                    \\\"$t=Get-Date -Format yyyyMMdd_HHmmss; \
                    $b='${BACKUP_TO}/'+$t; \
                    New-Item -ItemType Directory -Force -Path $b | Out-Null; \
                    robocopy ${APP_PATH}/Areas  $b/Areas  /E /R:1 /W:1; \
                    robocopy ${APP_PATH}/Models $b/Models /E /R:1 /W:1; \
                    robocopy ${APP_PATH}/Views  $b/Views  /E /R:1 /W:1; \
                    robocopy ${APP_PATH}/bin    $b/bin    /E /R:1 /W:1\\\""
                    '''
                }
            }
        }

        stage('Delete Old Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no administrator@${TARGET_IP} \
                    "powershell -NoProfile -Command \\\"Remove-Item ${APP_PATH}/* -Recurse -Force\\\""
                    '''
                }
            }
        }

        stage('Deploy New Files (SCP)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    scp -o StrictHostKeyChecking=no -r Areas Models Views bin \
                    administrator@${TARGET_IP}:${APP_PATH}/
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no administrator@${TARGET_IP} \
                    "powershell -NoProfile -Command \\\"Get-ChildItem ${APP_PATH}\\\""
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ DEPLOYMENT SUCCESS — backup + deploy aman'
        }
        failure {
            echo '❌ DEPLOYMENT FAILED — backup masih aman, siap rollback'
        }
        always {
            cleanWs()
        }
    }
}
