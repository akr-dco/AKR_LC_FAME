pipeline {
    agent any

    environment {
        WIN_HOST = "192.168.192.131"
        WIN_USER = "administrator"
        TARGET_DIR = "C:/inetpub/wwwroot/AKR_LC_FAME"
        BACKUP_DIR = "E:/BACKUP/AFTER/AKR_LC_FAME"
        SSH_CRED  = "ssh-jenkinsprod"
    }

    stages {

        stage('Backup Existing Files') {
    steps {
        sshagent(credentials: [env.SSH_CRED]) {
            sh '''
ENCODED=$(cat <<'EOF' | iconv -t UTF-16LE | base64 -w 0
$ts = Get-Date -Format yyyyMMdd_HHmmss
$dst = "E:/BACKUP/AFTER/AKR_LC_FAME$ts"

New-Item -ItemType Directory -Force -Path $dst | Out-Null

$folders = @("Areas","Models","Views","bin")

foreach ($f in $folders) {
    $src = "C:/inetpub/wwwroot/AKR_LC_FAME/$f"
    if (Test-Path $src) {
        robocopy $src "$dst\\$f" /E /R:1 /W:1 | Out-Null
    }
}

Write-Host "BACKUP OK => $dst"
EOF
)

ssh -o StrictHostKeyChecking=no administrator@192.168.192.131 \
powershell -NoProfile -EncodedCommand $ENCODED
'''
        }
    }
}

        stage('Deploy New Files') {
    steps {
        sshagent(credentials: [env.SSH_CRED]) {
            sh '''
# === DELETE OLD FOLDERS (SYNC WITH GIT) ===
ENCODED=$(echo "
\$folders = @('Areas','Models','Views','bin')
foreach (\$f in \$folders) {
    \$path = '${TARGET_DIR}\\\\' + \$f
    if (Test-Path \$path) {
        Remove-Item -Recurse -Force \$path
    }
}
Write-Host 'OLD FOLDERS DELETED'
" | iconv -t UTF-16LE | base64 -w 0)

ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} \
powershell -NoProfile -EncodedCommand $ENCODED

# === COPY NEW FILES FROM JENKINS WORKSPACE ===
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
cmd /c "dir C:\\inetpub\\wwwroot\\AKR_LC_FAME"
'''
                }
            }
        }
    }

    post {
        success {
            echo "✅ DEPLOY SUCCESS"
        }
        failure {
            echo "❌ DEPLOY FAILED — BACKUP MASIH AMAN"
        }
        always {
            cleanWs()
        }
    }
}
