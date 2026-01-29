pipeline {
    agent any

    environment {
        SSH_CRED = 'ssh-jenkinsprod'
        TARGET   = 'administrator@192.168.192.131'
        APPDIR   = 'C:/inetpub/wwwroot/AKR_LC_FAME'
        BACKUP   = 'E:/BACKUP/AFTER'
    }

    stages {

        stage('Backup Existing Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${TARGET} ^
                    "cmd /c ^
                    set TS=%DATE:~-4%%DATE:~4,2%%DATE:~7,2%_%TIME:~0,2%%TIME:~3,2%%TIME:~6,2% ^
                    & set TS=%TS: =0% ^
                    & mkdir ${BACKUP}\\%TS% ^
                    & robocopy ${APPDIR}\\Areas  ${BACKUP}\\%TS%\\Areas  /E /R:1 /W:1 ^
                    & robocopy ${APPDIR}\\Models ${BACKUP}\\%TS%\\Models /E /R:1 /W:1 ^
                    & robocopy ${APPDIR}\\Views  ${BACKUP}\\%TS%\\Views  /E /R:1 /W:1 ^
                    & robocopy ${APPDIR}\\bin    ${BACKUP}\\%TS%\\bin    /E /R:1 /W:1"
                    '''
                }
            }
        }

        stage('Delete Old Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${TARGET} ^
                    "cmd /c rmdir /S /Q ${APPDIR}"
                    '''
                }
            }
        }

        stage('Deploy New Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    scp -o StrictHostKeyChecking=no -r Areas Models Views bin \
                    ${TARGET}:${APPDIR}/
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${TARGET} ^
                    "cmd /c dir ${APPDIR}"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ DEPLOYMENT SUCCESS — backup & deploy aman'
        }
        failure {
            echo '❌ DEPLOYMENT FAILED — backup masih aman'
        }
        always {
            cleanWs()
        }
    }
}
