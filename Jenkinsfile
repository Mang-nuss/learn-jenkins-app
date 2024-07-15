pipeline {
    agent any

    //for deployment
    environment {
        NETLIFY_SITE_ID = 'fd1215fe-8aca-440f-bad4-22f450f88ecb'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    
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
        /*
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
                

                sh '''
                    npx playwright install
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test
                '''
                
                //The server installaton can be done without the -g (global) tag
            }
        }
        */

        stage('Run Tests') {
            parallel {

                stage('unit tests') {
                
                    agent {
                        docker {
                            image 'node:18-alpine'   
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            test -f build/index.html
                            ls build/index*
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }
                /*
                stage('E2E test') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            #npx playwright install
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test
                        '''
                    }
                } 
                */
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'   
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "deploying to STAGING site with ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }
        
        stage('Approval') {
            steps {
                input message: "Do you want to deploy to prod?",
                input ok button: "Yep, I do!"
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-alpine'   
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "deploying to PRODUCTION site with ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        
        stage('Post Prod E2E test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    #npx playwright install
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test
                '''
            }
        } 
    }
    /*
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
    */
}