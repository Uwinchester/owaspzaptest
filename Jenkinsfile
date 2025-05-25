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
                sh 'docker rm -f pfa_app'
                sh "docker run -d -p 9090:9090 --name pfa_app uwinchester/pfa_app"
            }
        }
        stage('DAST') {
            steps{
                script {
                    sh '''
                        docker pull zaproxy/zap-stable
                        docker run --rm \
                            -t zaproxy/zap-stable \
                            zap-baseline.py \
                            -t http://104.248.252.219:9090/
                        '''
                }
            }
        }
    }
}
