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
                /*
                sh '''
                    echo "This time with Docker"
                    ls -latr
                    touch container-yeah.txt
                    npm --version
                    node --version
                    npm ci
                    npm run build
                '''
                */
                sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test
                '''
                //The server installaton can be done without the -g (global) tag
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
    /*
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
    */
}