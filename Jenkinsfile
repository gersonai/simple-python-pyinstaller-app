pipeline {
    agent none
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') { 
            agent {
                docker {
                    image 'python:3-alpine' 
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py' 
                stash(name: 'compiled-results', includes: 'sources/*.py*') 
            }
        }
	stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
	stage('Deliver') {
            agent any
            environment {
                VOLUME = '/Users/gerson/myDockers/jenkins/simple-python-pyinstaller-app/sources:/src'
                IMAGE = 'cdrx/pyinstaller-linux:python3'
            }
            steps {
                sh "docker run --rm -w /src -v ${VOLUME} ${IMAGE} 'pyinstaller -F add2vals.py'"
            }
        }
    }
}