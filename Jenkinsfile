pipeline {
    agent none
    environment {
        AWS_ACCESS_KEY_ID     = credentials('986117744524').AWS_ACCESS_KEY_ID.toString()
        AWS_SECRET_ACCESS_KEY = credentials('986117744524').AWS_SECRET_ACCESS_KEY.toString()

        S3_BUCKET             = 'qasim-resume-website'
        SERVER_IP             = '54.196.61.65'
        SERVER_USER           = 'root'
        SERVER_DESTINATION    = '/opt/calc'
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
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
            agent {
                docker {
                    image 'cdrx/pyinstaller-linux:python2'
                }
            }
            steps {
                sh 'pyinstaller --onefile sources/add2vals.py'
            }
            post {
                success {
                    archiveArtifacts 'dist/add2vals'
                }
            }
        }
        stage('Upload to S3') {
            agent any
            steps {
                script {
                    withAWS(region: 'us-east-1', credentials: 'AKIA6LGJSU6GMNQT4RPB') {
                        s3Upload(bucket: S3_BUCKET, workingDir: 'dist', includePathPattern: '**/*')
                    }
                }
            }
        }
        stage('Deploy to Server') {
            agent any
            steps {
                script {
                    sshagent(['AKIA6LGJSU6GMNQT4RPB']) {
                        sh "scp -o StrictHostKeyChecking=no -r dist/add2vals ${SERVER_USER}@${SERVER_IP}:${SERVER_DESTINATION}"
                    }
                }
            }
        }
    }
}
