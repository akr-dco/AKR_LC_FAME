pipeline {
    agent any

    environment {
        WIN_HOST   = '192.168.192.131'
        WIN_USER   = 'administrator'
        TARGET_DIR = 'C:/inetpub/wwwroot/AKR_LC_FAME'
        BACKUP_DIR = 'E:/BACKUP/AFTER'
        SSH_CRED   = 'ssh-jenkinsprod'
    }

    stages {

        stage('Backup Existing Files (CMD)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} "cmd /c ^
set TS=%DATE:~-4%%DATE:~4,2%%DATE:~7,2%_%TIME:~0,2%%TIME:~3,2%%TIME:~6,2% ^
&& set TS=%TS: =0% ^
&& mkdir ${BACKUP_DIR}\\%TS% ^
&& robocopy ${TARGET_DIR}\\Areas  ${BACKUP_DIR}\\%TS%\\Areas  /E /R:1 /W:1 ^
&& robocopy ${TARGET_DIR}\\Models ${BACKUP_DIR}\\%TS%\\Models /E /R:1 /W:1 ^
&& robocopy ${TARGET_DIR}\\Views  ${BACKUP_DIR}\\%TS%\\Views  /E /R:1 /W:1 ^
&& robocopy ${TARGET_DIR}\\bin    ${BACKUP_DIR}\\%TS%\\bin    /E /R:1 /W:1 ^
|| exit 0"
'''
                }
            }
        }

        stage('Delete Old Files (CMD)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} "cmd /c ^
rmdir /S /Q ${TARGET_DIR}\\Areas ^
& rmdir /S /Q ${TARGET_DIR}\\Models ^
& rmdir /S /Q ${TARGET_DIR}\\Views ^
& rmdir /S /Q ${TARGET_DIR}\\bin"
'''
                }
            }
        }

        stage('Deploy New Files (SCP)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
scp -o StrictHostKeyChecking=no -r Areas Models Views bin \
${WIN_USER}@${WIN_HOST}:${TARGET_DIR}/
'''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} "cmd /c dir ${TARGET_DIR}"
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
            echo '❌ DEPLOYMENT FAILED — backup masih aman, bisa rollback'
        }
        always {
            cleanWs()
        }
    }
}
