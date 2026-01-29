pipeline {
    agent any

    environment {
        SSH_CRED   = 'ssh-jenkinsprod'
        WIN_USER   = 'administrator'
        WIN_HOST   = '192.168.192.131'
        APP_DIR    = 'C:\\inetpub\\wwwroot\\AKR_LC_FAME'
        BACKUP_DIR = 'E:\\BACKUP\\AFTER'
    }

    stages {

        stage('Backup Existing Files') {
    steps {
        sshagent(credentials: [env.SSH_CRED]) {
            sh '''
ssh -o StrictHostKeyChecking=no administrator@192.168.192.131 powershell -NoProfile -Command '
$ts = Get-Date -Format yyyyMMdd_HHmmss
$dst = Join-Path "E:\\BACKUP\\AFTER" $ts
New-Item -ItemType Directory -Force -Path $dst | Out-Null

foreach ($f in "Areas","Models","Views","bin") {
    $src = Join-Path "C:\\inetpub\\wwwroot\\AKR_LC_FAME" $f
    if (Test-Path $src) {
        robocopy $src (Join-Path $dst $f) /E /R:1 /W:1 | Out-Null
    }
}

Write-Host "Backup OK => $dst"
'
'''
        }
    }
}

        stage('Deploy New Files (SCP)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh """
scp -o StrictHostKeyChecking=no -r \
Areas Models Views bin \
${WIN_USER}@${WIN_HOST}:${APP_DIR}/
"""
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh """
ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} \
"cmd /c dir \\"C:\\inetpub\\wwwroot\\AKR_LC_FAME\\""
"""
                }
            }
        }
    }

    post {
        success {
            echo '✅ BACKUP OK — DEPLOY OK'
        }
        failure {
            echo '❌ FAILED — FILE ASLI MASIH ADA DI BACKUP'
        }
        always {
            cleanWs()
        }
    }
}
