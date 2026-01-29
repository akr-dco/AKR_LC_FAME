pipeline {
    agent any

    environment {
        WIN_HOST  = "192.168.192.131"
        WIN_USER  = "administrator"
        SSH_CRED = "ssh-jenkinsprod"

        APP_NAME  = "AKR_LC_FAME"
        TARGET_DIR = "C:/inetpub/wwwroot/AKR_LC_FAME"
        BACKUP_ROOT = "E:/BACKUP/AFTER"
    }

    stages {

        stage('Backup Existing Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -EncodedCommand "JAB0AGkAbQBlAHMAdABhAG0AcAAgAD0AIABHAGUAdAAtAEQAYQB0AGUAIAAtAEYAbwByAG0AYQB0ACAAJwB5AHkAeQB5AE0ATQBkAGQALQBIAGgAbQBtAHMAcwAnAAoAJABiAGEAYwBrAHUAcABSAG8AbwB0ACAAPQAgACcARQA6AFwAQgBBAEMASwBVAFwAQQBGAEUAUgAnAAoAJABiAGEAYwBrAHUAcABQAGEAdABoACAAPQAgAEoAbwBpAG4ALQBQAGEAdABoACAAJABiAGEAYwBrAHUAcABSAG8AbwB0ACAAJAB0AGkAbQBlAHMAdABhAG0AcAAKACQAdABhAHIAZwBlAHQAIAA9ACAAJwBDADoAXABpAG4AZQB0AHAAdQBiAFwAdwB3AHcAcgBvAG8AdABcAEEASwBSAF8ATABDAF8ARgBBAE0ARQAnAAoATgBlAHcALQBJAHQAZQBtACAALQBJAHQAZQBtAFQAeQBwAGUAIABEAGkAcgBlAGMAdABvAHIAeQAgAC0ARgBvAHIAYwBlACAALQBQAGEAdABoACAAJABiAGEAYwBrAHUAcABQAGEAdABoACAAfAAgAE8AdQB0AC0ATgB1AGwAbAAKAGYAbwByAGUAYQBjAGgAIAAoACQAZgBvAGwAZABlAHIAIABpAG4AIAAnAEEAcgBlAGEAcwAnACwAJwBNAG8AZABlAGwAcwAnACwAJwBWAGkAZQB3AHMAJwAsACcAYgBpAG4AJwApACAAewAKACAAIAAkAHMAcgBjACAAPQAgAEoAbwBpAG4ALQBQAGEAdABoACAAJAB0AGUAcgBnAGUAdAAgACQAZgBvAGwAZABlAHIAAAoAIAAgACQAZABzAHQAIAA9ACAAAEoAbwBpAG4ALQBQAGEAdABoACAAJABiAGEAYwBrAHUAcABQAGEAdABoACAAJABmAG8AbABkAGUAcgAKACAAIABpAGYAIABUAGUAcwB0AC0AUABhAHQAaAAgACQAcwByAGMAKQAgAHsACgAgACAAIAAgAHIAbwBiAG8AYwBvAHAAeQAgACQAcwByAGMAIAAkAGQAcwB0ACAALwBFACAALwBDADoAUwBBAFQAIAAvAFIALwAyACAALwBXADoAMgAgAHwAIABPAHUAdAAtAE4AdQBsAGwACgAgACAAfQAKAH0ACgBXAHIAaQB0AGUALQBIAG8AcwB0ACAAJwBCAGEAYwBrAHUAcAAgAGMAbwBtAHAAbABlAHQAZQBkADoAJwAgACQAYgBhAGMAawB1AHAAUABhAHQAaAA="
                    '''
                }
            }
        }

        stage('Delete Old Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -Command "
                    Remove-Item -Recurse -Force `
                        ${TARGET_DIR}\\Areas,
                        ${TARGET_DIR}\\Models,
                        ${TARGET_DIR}\\Views,
                        ${TARGET_DIR}\\bin `
                        -ErrorAction SilentlyContinue
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
                        AKR_LC_FAME/Areas \
                        AKR_LC_FAME/Models \
                        AKR_LC_FAME/Views \
                        AKR_LC_FAME/bin \
                        ${WIN_USER}@${WIN_HOST}:${TARGET_DIR}/
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -Command "
                    Test-Path ${TARGET_DIR}\\bin
                    "
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ DEPLOYMENT SUCCESS — backup tersedia di ${BACKUP_ROOT}"
        }
        failure {
            echo "❌ DEPLOYMENT FAILED — backup masih aman, siap rollback"
        }
        always {
            cleanWs()
        }
    }
}
