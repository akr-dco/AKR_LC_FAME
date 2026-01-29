pipeline {
    agent any

    environment {
        SSH_CRED  = 'ssh-jenkinsprod'
        WIN_USER  = 'administrator'
        WIN_HOST  = '192.168.192.131'
        TARGET_DIR = 'C:/inetpub/wwwroot/AKR_LC_FAME'
        BACKUP_ROOT = 'E:/BACKUP/AFTER'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Backup Existing Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -Command "
                        \$timestamp = Get-Date -Format 'yyyyMMdd_HHmmss'
                        \$backupPath = Join-Path '${BACKUP_ROOT}' \$timestamp
                        \$target = '${TARGET_DIR}'

                        New-Item -ItemType Directory -Force -Path \$backupPath | Out-Null

                        foreach (\$folder in 'Areas','Models','Views','bin') {
                            \$src = Join-Path \$target \$folder
                            \$dst = Join-Path \$backupPath \$folder
                            if (Test-Path \$src) {
                                robocopy \$src \$dst /E /R:2 /W:2 | Out-Null
                            }
                        }

                        Write-Host 'Backup completed to:' \$backupPath
                    "
                    """
                }
            }
        }

        stage('Delete Old Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -Command "
                        foreach (\$folder in 'Areas','Models','Views','bin') {
                            \$path = Join-Path '${TARGET_DIR}' \$folder
                            if (Test-Path \$path) {
                                Remove-Item \$path -Recurse -Force
                            }
                        }
                        Write-Host 'Old folders deleted'
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
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -Command "
                        Get-ChildItem '${TARGET_DIR}'
                    "
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ DEPLOYMENT SUCCESS — backup aman & versi baru live'
        }
        failure {
            echo '❌ DEPLOYMENT FAILED — backup masih aman, siap rollback'
        }
        always {
            cleanWs()
        }
    }
}
