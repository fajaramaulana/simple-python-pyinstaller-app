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
            /* groovylint-disable-next-line DuplicateStringLiteral */
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
            python sources/add2vals.py 10 20
            sleep 60
            '''
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
