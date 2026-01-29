pipeline {
    agent any

    environment {
        WIN_HOST = "192.168.192.131"
        WIN_USER = "administrator"
        SSH_CRED = "ssh-jenkinsprod"
        TARGET_DIR = "C:/inetpub/wwwroot/AKR_LC_FAME"
        SRC_DIR = "AKR_LC_FAME"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Clean Target Folder (Windows)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} \
                    "powershell -NoProfile -Command \\
                    if (Test-Path '${TARGET_DIR}/Areas') { Remove-Item -Recurse -Force '${TARGET_DIR}/Areas' }; \
                    if (Test-Path '${TARGET_DIR}/Models') { Remove-Item -Recurse -Force '${TARGET_DIR}/Models' }; \
                    if (Test-Path '${TARGET_DIR}/Views') { Remove-Item -Recurse -Force '${TARGET_DIR}/Views' }; \
                    if (Test-Path '${TARGET_DIR}/bin') { Remove-Item -Recurse -Force '${TARGET_DIR}/bin' }"
                    """
                }
            }
        }

        stage('Copy New Files (SCP)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh """
                    scp -o StrictHostKeyChecking=no -r ${SRC_DIR}/* \
                    ${WIN_USER}@${WIN_HOST}:${TARGET_DIR}/
                    """
                }
            }
        }

        stage('Verify Files (Optional)') {
    steps {
        sshagent(credentials: [env.SSH_CRED]) {
            sh """
            ssh -o StrictHostKeyChecking=no administrator@192.168.192.131 \
            "powershell -NoProfile -Command \\
            Get-ChildItem 'C:/inetpub/wwwroot/AKR_LC_FAME' | Select-Object Name"
            """
        }
    }
}

    post {
        success {
            echo "✅ Deployment sukses tanpa restart IIS"
        }
        failure {
            echo "❌ Deployment gagal — cek stage error"
        }
    }
}