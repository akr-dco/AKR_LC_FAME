pipeline {
    agent any

    environment {
        WIN_HOST   = '192.168.192.131'
        WIN_USER   = 'administrator'
        TARGET_DIR = 'C:/inetpub/wwwroot/AKR_LC_FAME'
        BACKUP_DIR = 'E:/BACKUP/AFTER'
        SSH_CRED   = 'akr-dco'
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Verify Workspace') {
            steps {
                sh '''
                echo "📂 Jenkins workspace:"
                pwd
                ls -lah
                '''
            }
        }

        stage('Backup Existing Files (Versioning)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} \
                    powershell -NoProfile -Command "& {
                        $timestamp  = Get-Date -Format 'yyyyMMdd-HHmmss'
                        $backupRoot = 'E:\\BACKUP\\AFTER'
                        $backupPath = Join-Path $backupRoot $timestamp
                        $target     = 'C:\\inetpub\\wwwroot\\AKR_LC_FAME'

                        New-Item -ItemType Directory -Force -Path $backupPath | Out-Null

                        foreach ($folder in 'Areas','Models','Views','bin') {
                            $src = Join-Path $target $folder
                            $dst = Join-Path $backupPath $folder

                            if (Test-Path $src) {
                                robocopy $src $dst /E /COPY:DAT /R:2 /W:2 | Out-Null
                            }
                        }

                        Write-Host '✅ Backup completed:' $backupPath
                    }"
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
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} \
                    powershell -NoProfile -Command "& {
                        Get-ChildItem 'C:\\inetpub\\wwwroot\\AKR_LC_FAME' |
                        Select-Object Name, LastWriteTime
                    }"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ DEPLOYMENT SUCCESS'
        }

        failure {
            echo '❌ DEPLOYMENT FAILED — backup tersedia untuk rollback'
        }

        always {
            cleanWs()
        }
    }
}
