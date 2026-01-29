pipeline {
    agent any

    environment {
        WIN_HOST  = "192.168.192.131"
        WIN_USER  = "administrator"
        SSH_CRED  = "ssh-jenkinsprod"

        TARGET_DIR = "C:/inetpub/wwwroot/AKR_LC_FAME"
        BACKUP_DIR = "E:/BACKUP/AFTER"
    }

    stages {

        stage('Backup Existing Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    POWERSHELL_SCRIPT='
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupPath = Join-Path "${BACKUP_DIR}" $timestamp

New-Item -ItemType Directory -Force -Path $backupPath | Out-Null

$folders = @("Areas","Models","Views","bin")

foreach ($folder in $folders) {
    $src = Join-Path "${TARGET_DIR}" $folder
    $dst = Join-Path $backupPath $folder

    if (Test-Path $src) {
        robocopy $src $dst /E /R:2 /W:2 /NFL /NDL /NJH /NJS | Out-Null
    }
}

Write-Host "Backup completed to $backupPath"
'

ENCODED=$(echo "$POWERSHELL_SCRIPT" | iconv -t UTF-16LE | base64 -w 0)

ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -EncodedCommand $ENCODED
                    '''
                }
            }
        }

        stage('Delete Old Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -Command "
                    Remove-Item ${TARGET_DIR}\\Areas  -Recurse -Force -ErrorAction SilentlyContinue
                    Remove-Item ${TARGET_DIR}\\Models -Recurse -Force -ErrorAction SilentlyContinue
                    Remove-Item ${TARGET_DIR}\\Views  -Recurse -Force -ErrorAction SilentlyContinue
                    Remove-Item ${TARGET_DIR}\\bin    -Recurse -Force -ErrorAction SilentlyContinue
                    "
                    '''
                }
            }
        }

        stage('Deploy New Files (SCP)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    scp -o StrictHostKeyChecking=no -r \
                        Areas \
                        Models \
                        Views \
                        bin \
                        ${WIN_USER}@${WIN_HOST}:${TARGET_DIR}/
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -Command "
                    Get-ChildItem ${TARGET_DIR}
                    "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ DEPLOYMENT SUCCESS — backup + deploy aman"
        }
        failure {
            echo "❌ DEPLOYMENT FAILED — backup masih aman, siap rollback"
        }
        always {
            cleanWs()
        }
    }
}
