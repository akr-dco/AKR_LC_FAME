pipeline {
    agent any

    environment {
        WIN_HOST   = '192.168.192.131'
        WIN_USER   = 'administrator'
        TARGET_DIR = 'C:/inetpub/wwwroot/AKR_LC_FAME'
        BACKUP_DIR = 'E:/BACKUP/AFTER'
        SSH_CRED   = 'ssh-jenkinsprod'
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
                echo "📂 Workspace:"
                pwd
                ls -lah
                '''
            }
        }

        stage('Backup Existing Files (Versioning)') {
            steps {
                sshagent(credentials: [env.SSH_CRED]) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -EncodedCommand "
                    JAB0AGkAbQBlAHMAdABhAG0AcAAgAD0AIABHAGUAdAAtAEQAYQB0AGUAIAAtAEYAbwByAG0AYQB0ACAAJwB5AHkAeQB5AE0ATQBkAGQALQBIAGgAbQBtAHMAcwAnAAoAJABiAGEAYwBrAHUAcABSAG8AbwB0ACAAPQAgACcARQA6AFwAQgBBAEMASwBVAFwAQQBGAEUAUgAnAAoAJABiAGEAYwBrAHUAcABQAGEAdABoACAAPQAgAEoAbwBpAG4ALQBQAGEAdABoACAAJABiAGEAYwBrAHUAcABSAG8AbwB0ACAAJAB0AGkAbQBlAHMAdABhAG0AcAAKACQAdABhAHIAZwBlAHQAIAA9ACAAJwBDADoAXABpAG4AZQB0AHAAdQBiAFwAdwB3AHcAcgBvAG8AdABcAEEASwBSAF8ATABDAF8ARgBBAE0ARQAnAAoATgBlAHcALQBJAHQAZQBtACAALQBJAHQAZQBtAFQAeQBwAGUAIABEAGkAcgBlAGMAdABvAHIAeQAgAC0ARgBvAHIAYwBlACAALQBQAGEAdABoACAAJABiAGEAYwBrAHUAcABQAGEAdABoACAAfAAgAE8AdQB0AC0ATgB1AGwAbAAKAGYAbwByAGUAYQBjAGgAIAAoACQAZgBvAGwAZABlAHIAIABpAG4AIAAnAEEAcgBlAGEAcwAnACwAJwBNAG8AZABlAGwAcwAnACwAJwBWAGkAZQB3AHMAJwAsACcAYgBpAG4AJwApACAAewAKACAAIAAkAHMAcgBjACAAPQAgAEoAbwBpAG4ALQBQAGEAdABoACAAJAB0AGUAcgBnAGUAdAAgACQAZgBvAGwAZABlAHIAAAoAIAAgACQAZABzAHQAIAA9ACAAAEoAbwBpAG4ALQBQAGEAdABoACAAJABiAGEAYwBrAHUAcABQAGEAdABoACAAJABmAG8AbABkAGUAcgAKACAAIABpAGYAIABUAGUAcwB0AC0AUABhAHQAaAAgACQAcwByAGMAKQAgAHsACgAgACAAIAAgAHIAbwBiAG8AYwBvAHAAeQAgACQAcwByAGMAIAAkAGQAcwB0ACAALwBFACAALwBDADoAUwBBAFQAIAAvAFIALwAyACAALwBXADoAMgAgAHwAIABPAHUAdAAtAE4AdQBsAGwACgAgACAAfQAKAH0ACgBXAHIAaQB0AGUALQBIAG8AcwB0ACAAJwBCAGEAYwBrAHUAcAAgAGMAbwBtAHAAbABlAHQAZQBkADoAJwAgACQAYgBhAGMAawB1AHAAUABhAHQAaAA="
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
                    ssh -o StrictHostKeyChecking=no ${WIN_USER}@${WIN_HOST} powershell -NoProfile -EncodedCommand "
                    RwBlAHQALQBDAGgAaQBsAGQASQB0AGUAbQAgACcAQwA6AFwAaQBuAGUAdABwAHUAYgBcAHcAdwB3AHIAbwBvAHQAXABBAEsAUgBfAEwAQwBfAEYAQQBNAEUAJwAgAHwAIABTAGUAbABlAGMAdAAtAE8AYgBqAGUAYwB0ACAATgBhAG0AZQAsACAATABhAHMAdABXAHIAaQB0AGUAVABpAG0AZQA="
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
            echo '❌ DEPLOYMENT FAILED — backup tersedia'
        }
        always {
            cleanWs()
        }
    }
}
