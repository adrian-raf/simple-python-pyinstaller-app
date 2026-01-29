pipeline {
    agent {
        docker { 
            image 'python:3.14' 
            args '-u root:root'
        }
    }
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            steps {
                sh 'python3 -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            steps {
                sh 'pip install pytest'
                sh 'pytest --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') { 
            steps {
                sh 'pip install pyinstaller'
                sh "pyinstaller --onefile sources/add2vals.py" 
            }
            post {
                success {
                    archiveArtifacts 'dist/add2vals' 
                }
            }
        }
        
        stage('Manual Approval') {
            steps {
                input message: 'Lanjutkan ke tahap Deploy?'
            }
        }

        stage('Deploy') {
            steps {
                sh './dist/add2vals & echo $! > app.pid'
                sleep(time: 1, unit: 'MINUTES')
                sh 'kill $(cat app.pid) || true'
            }
        }
    }
}