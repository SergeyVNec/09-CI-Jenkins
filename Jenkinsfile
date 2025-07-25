pipeline {
    agent any

    environment {
        HOST_USER = "alkat"
        HOST_IP = "192.168.128.27"            // IP твоего Windows-хоста в сети
        VAGRANT_DIR = "c:/Users/alkat/Documents/"
        VM_USER = "vagrant"
    }

    stages {
        stage('Create VM via Vagrant on Host') {
            steps {
                // Запустить Vagrant на Windows-хосте по SSH
                sh """
                ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${HOST_USER}@${HOST_IP} 'cd "${VAGRANT_DIR}" && vagrant up --provider=virtualbox'
                """
            }
        }
        stage('Get VM SSH Info from Host') {
            steps {
                script {
                    // Получить путь к ключу и IP из Vagrant на хосте
                    def ssh_config = sh(
                        script: """
                        ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${HOST_USER}@${HOST_IP} 'cd "${VAGRANT_DIR}" && vagrant ssh-config'
                        """,
                        returnStdout: true
                    )
                    // Распарсить ssh_config, чтобы получить IP и путь к ключу
                    def host = (ssh_config =~ /HostName ([^\s]+)/)[0][1]
                    def port = (ssh_config =~ /Port ([^\s]+)/)[0][1]
                    def key = (ssh_config =~ /IdentityFile ([^\s]+)/)[0][1]
                    echo "VM Host: ${host}"
                    echo "VM Port: ${port}"
                    echo "VM Key: ${key}"

                    // Скопировать приватный ключ с Windows-хоста на Jenkins VM (временно для пайплайна)
                    //sh """
                    //ssh -i ${VAGRANT_DIR} -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -p ${port} ${VM_USER}@${host} 'type "${key}"' > temp_vagrant_key
                    //chmod 600 temp_vagrant_key
                    //"""
                    env.VM_IP = host
                    env.VM_PORT = port
                    env.VM_KEY = key
                    
                    sh """
                    scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${HOST_USER}@${HOST_IP}:"${env.VM_KEY}" ~/.ssh/temp_vagrant_key
                    chmod 600 ~/.ssh/temp_vagrant_key
                    """
                }
            }
        }
        stage('Install Apache via SSH from Jenkins') {
            steps {
                script {
                    sh """
                    ssh -i ~/.ssh/temp_vagrant_key -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${VM_USER}@192.168.56.111 \\
                    'sudo apt-get update && sudo apt-get install -y apache2 && sudo systemctl enable apache2 && sudo systemctl start apache2'
                    """
                }
            }
        }
        stage('Check 4xx/5xx in Apache Logs via SSH') {
            steps {
                script {
                    sh '''
                    ssh -i ~/.ssh/temp_vagrant_key -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${VM_USER}@192.168.56.111 "
                        curl -s http://localhost/page.php > /dev/null
                        if [ -f /var/log/apache2/access.log ]; then
                            grep -E '\\\" [45][0-9][0-9] ' /var/log/apache2/access.log || echo 'No 4xx/5xx errors found.'
                        else
                            echo 'Log file not found.'
                        fi
                    "
                    '''
                }
            }
        }
    }
    post {
        always {
            sh 'rm -f temp_vagrant_key'  // удаляем временный ключ
        }
    }
}
