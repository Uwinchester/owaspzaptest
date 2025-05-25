pipeline {
    agent any
    stages {

        stage ('build') {
            steps {
                echo 'Building the application...'
                sh "docker build -t uwinchester/pfa_app ."
            }
        }
        stage ('push') {
            steps {
                echo 'Pushing the image to dockerhub...'
                sh 'docker login -u uwinchester -p youdou203'
                sh 'docker push uwinchester/pfa_app'
            }
        }

        stage ('deploy to tomcat for DAST') {
            steps {
                echo 'deploying to tomcat'
                sh 'docker-compose down --rmi local --volumes --remove-orphans || true'
                sh 'docker rm -f uwinchester/pfa_app'
                sh "docker-compose up -d"
            }
        }
        stage('DAST') {
            steps{
                script {
                    sh '''
                        docker pull zaproxy/zap-stable
                        docker run --rm --name zap_scan \
                            -v ${WORKSPACE}:/zap/ \
                            -t zaproxy/zap-stable \
                            zap-baseline.py \
                            -t http://104.248.252.219:9090/ \
                            -r /zap/zap-report.html
                        '''
                    sh 'docker exec zap_scan ls -l /zap'
                    
                    
                }
                echo "[INFO] ZAP scan completed. Check the report if the build fails."
                archiveArtifacts 'zap-report.html'
            }
        }
    }
    post {
        always {
            archiveArtifacts 'zap-report.html'

            publishHTML target: [
                allowMissing: true,
                reportDir: '.',
                reportFiles: 'zap-report.html', 
                reportName: 'ZAP Report',
                keepAll: true
            ]
        }
    }
}
