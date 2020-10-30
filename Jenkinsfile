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
            steps {
                sh "docker run --rm -w /src -v '${(pwd)}:/src' 'cdrx/pyinstaller-linux:python3' 'pyinstaller -F sources/add2vals.py'"
            }
            post {
                success {
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
                    sh "docker run --rm -v '${(pwd)}:/src' 'cdrx/pyinstaller-linux:python3' 'rm -rf build dist'"
                }
            }
        }
    }
}