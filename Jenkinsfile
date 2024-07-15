pipeline {
    agent any
    
    stages {
        stage('w/o docker') {
            steps {
                sh '''
                    echo "Without Docker"
                    ls -latr
                    touch container-nope.txt
                '''
            }
        }
        
        //build with docker
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'   
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "This time with Docker"
                    ls -latr
                    touch container-yeah.txt
                    npm --version
                    node --version
                    npm ci
                    npm run build
                '''
            }
        }
        /*
        stage('Test') {
            steps {
                sh '''
                    test -f build/index.html
                    ls build/index*
                    npm test
                '''
            }
        }
        */
    }
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}