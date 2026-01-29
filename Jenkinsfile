pipeline {
    agent any

    environment {
        WIN_HOST   = "192.168.192.131"
        WIN_USER   = "administrator"
        TARGET_DIR = "C:/inetpub/wwwroot/AKR_LC_FAME"
        BACKUP_DIR = "E:/BACKUP/AFTER"
        SSH_CRED   = "ssh-jenkinsprod"
        TS         = "${new Date().format('yyyyMMdd_HHmmss')}"
    }

    stages {

        stage('Backup Existing Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -Command "
                        New-Item -ItemType Directory -Force -Path '${BACKUP_DIR}/${TS}' | Out-Null

                        \$folders = @('Areas','Models','Views','bin')
                        foreach (\$f in \$folders) {
                            if (Test-Path '${TARGET_DIR}/' + \$f) {
                                Copy-Item '${TARGET_DIR}/' + \$f '${BACKUP_DIR}/${TS}/' -Recurse -Force
                            }
                        }
                    "
                    """
                }
            }
        }

        stage('Delete Old Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -Command "
                        Remove-Item '${TARGET_DIR}/Areas'  -Recurse -Force -ErrorAction SilentlyContinue
                        Remove-Item '${TARGET_DIR}/Models' -Recurse -Force -ErrorAction SilentlyContinue
                        Remove-Item '${TARGET_DIR}/Views'  -Recurse -Force -ErrorAction SilentlyContinue
                        Remove-Item '${TARGET_DIR}/bin'    -Recurse -Force -ErrorAction SilentlyContinue
                    "
                    """
                }
            }
        }

        stage('Deploy New Files (SCP)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh """
                    scp -o StrictHostKeyChecking=no -r \
                        Areas \
                        Models \
                        Views \
                        bin \
                        ${WIN_USER}@${WIN_HOST}:${TARGET_DIR}/
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh """
                    ssh ${WIN_USER}@${WIN_HOST} powershell -Command "
                        Get-ChildItem '${TARGET_DIR}' | Select Name
                    "
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ DEPLOYMENT SUCCESS — backup tersimpan di ${BACKUP_DIR}/${TS}"
        }
        failure {
            echo "❌ DEPLOYMENT FAILED — backup masih aman, siap rollback"
        }
    }
}
