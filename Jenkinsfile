node {
    env.CI = 'true'
    
    try {
        checkout scm
        // Build Stage
        stage('Build') {
            docker.image('python:2-alpine').inside {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
            }
        }

        // Test Stage
        stage('Test') {
            docker.image('qnib/pytest').inside {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            junit 'test-reports/results.xml'  // Always publish test results
        }

        // Deliver Stage
        stage('Deliver') {
            docker.image('cdrx/pyinstaller-linux:python2').inside {
                sh 'pyinstaller --onefile sources/add2vals.py'
            }
            archiveArtifacts 'dist/add2vals'  // Archive the artifact after successful build
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
