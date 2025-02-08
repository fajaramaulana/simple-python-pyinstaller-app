node {
    env.CI = 'true'

    try {
        stage('Clone Repository') {
            checkout scm
        }

        stage('Prepare Environment') {
            docker.image('python:3.9-slim').inside {
                sh 'pwd'
                sh 'ls -l'
            }
        }

        stage('Build') {
            docker.image('python:3.9-slim').inside('--user root') {
                sh '''
                    python -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install pytest
                    python -m py_compile sources/add2vals.py sources/calc.py
                '''
            }
        }

        stage('Test') {
            docker.image('python:3.9-slim').inside('--user root') {
                sh '''
                    . venv/bin/activate
                    pytest --verbose sources/test_calc.py
                '''
            }
        }

        stage('Manual Approval') {
            input message: 'Lanjutkan ke tahap Deploy?'
        }

        stage('Deploy') {
            docker.image('python:3.9-slim').inside('--user root') {
                sh '''
                    . venv/bin/activate
                    python sources/add2vals.py 55 10
                    sleep 60
                '''
                
                echo 'Building executable file using PyInstaller...'
                sh '''
                    pip install pyinstaller &&
                    pyinstaller --onefile sources/add2vals.py &&
                    ls -l dist
                '''
                
                echo 'Transferring the executable file to the remote instance...'
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'second-instance-ssh-key',
                                      keyFileVariable: 'DEPLOY_KEY',
                                      usernameVariable: 'DEPLOY_USER'),
                    string(credentialsId: 'SECOND_INSTANCE_IP', variable: 'DEPLOY_HOST')
                ]) {
                    sh '''
                        # Membuat folder tujuan di instance kedua (contoh: /home/ubuntu/transfer)
                        ssh -i "$DEPLOY_KEY" -o StrictHostKeyChecking=no ${DEPLOY_USER}@$DEPLOY_HOST "mkdir -p /home/ubuntu/transfer"
                        # Mentransfer file executable. Pastikan file yang dihasilkan bernama "add2vals.exe"
                        scp -i "$DEPLOY_KEY" -o StrictHostKeyChecking=no dist/add2vals.exe ${DEPLOY_USER}@$DEPLOY_HOST:/home/ubuntu/transfer/
                    '''
                }
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        echo "Build failed: ${e.message}"
        throw e
    }
}
