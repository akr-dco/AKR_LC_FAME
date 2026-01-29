pipeline {
    agent any

    environment {
        WIN_HOST   = '192.168.192.131'
        WIN_USER   = 'administrator'
        TARGET_DIR = 'C:/inetpub/wwwroot/AKR_LC_FAME'
        SSH_CRED   = 'akr-dco'
    }

    stages {

        /* ================================
           BACKUP (VERSIONING)
        ================================= */
        stage('Backup Existing Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -EncodedCommand JAB0AGkAbQBlAHMAdABhAG0AcAAgAD0AIABHAGUAdAAtAEQAYQB0AGUAIAAtAEYAbwByAG0AYQB0ACAAIgB5AHkAeQB5AE0ATQBkAGQALQBIAGgAbQBtAHMAcwAiAAoAJABiAGEAYwBrAHUAcABSAG8AbwB0ACAAPQAgACIARQA6AFwAQgBBAEMASwBVAFwAQQBGAEUAUgAiAAoAJABiAGEAYwBrAHUAcABQAGEAdABoACAAPQAgAEoAbwBpAG4ALQBQAGEAdABoACAAJABiAGEAYwBrAHUAcABSAG8AbwB0ACAAJAB0AGkAbQBlAHMAdABhAG0AcAAKACQAdABhAHIAZwBlAHQAIAA9ACAAIgBDADoAXABpAG4AZQB0AHAAdQBiAFwAdwB3AHcAcgBvAG8AdABcAEEASwBSAF8ATABDAF8ARgBBAE0ARQAiAAoATgBlAHcALQBJAHQAZQBtACAALQBJAHQAZQBtAFQAeQBwAGUAIABEAGkAcgBlAGMAdABvAHIAeQAgAC0ARgBvAHIAYwBlACAALQBQAGEAdABoACAAJABiAGEAYwBrAHUAcABQAGEAdABoACAAfAAgAE8AdQB0AC0ATgB1AGwAbAAKAGYAbwByAGUAYQBjAGgAIAAoACQAZgBvAGwAZABlAHIAIABpAG4AIAAnAEEAcgBlAGEAcwAnACwAJwBNAG8AZABlAGwAcwAnACwAJwBWAGkAZQB3AHMAJwAsACcAYgBpAG4AJwApACAAewAKACAAIAAkAHMAcgBjACAAPQAgAEoAbwBpAG4ALQBQAGEAdABoACAAJAB0AGUAcgBnAGUAdAAgACQAZgBvAGwAZABlAHIAAAoAIAAgACQAZABzAHQAIAA9ACAAAEoAbwBpAG4ALQBQAGEAdABoACAAJABiAGEAYwBrAHUAcABQAGEAdABoACAAJABmAG8AbABkAGUAcgAKACAAIABpAGYAIABcACgAVABlAHMAdAAtAFAAYQB0AGgAIAAkAHMAcgBjACkAIAB7AAoAIAAgACAAIAByAG8AYgBvAGMAbwBwAHkAIAAkAHMAcgBjACAAPQAgACQAZABzAHQAIAAvAEUAIAAvAEMATwBQAFkAOgBEAEEAVAAgAC8AUgA6ADIAIAAvAFcAOgAyACAAfAAgAE8AdQB0AC0ATgB1AGwAbAAKACAAIAB9AAoAfQAKAFcAcgBpAHQAZQAtAEgAbwBzAHQAIAAiAEIAYQBjAGsAdQBwACAAYwBvAG0AcABsAGUAdABlAGQAIg==
'''
                }
            }
        }

        /* ================================
           DELETE OLD FILES
        ================================= */
        stage('Delete Old Files') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -Command "
$target='${TARGET_DIR}'
foreach ($f in 'Areas','Models','Views','bin') {
    $p = Join-Path $target $f
    if (Test-Path $p) {
        Remove-Item $p -Recurse -Force
    }
}
Write-Host 'Old files deleted'
"
'''
                }
            }
        }

        /* ================================
           DEPLOY NEW FILES
        ================================= */
        stage('Deploy New Files (SCP)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
scp -o StrictHostKeyChecking=no -r \
    Areas Models Views bin \
    ${WIN_USER}@${WIN_HOST}:${TARGET_DIR}/
'''
                }
            }
        }

        /* ================================
           VERIFY
        ================================= */
        stage('Verify Deployment') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -Command "
Get-ChildItem '${TARGET_DIR}' | Select Name, LastWriteTime
"
'''
                }
            }
        }
    }

    post {
        success {
            echo '✅ DEPLOYMENT SUCCESS — backup tersedia'
        }
        failure {
            echo '❌ DEPLOYMENT FAILED — backup masih aman, siap rollback'
        }
        always {
            cleanWs()
        }
    }
}
