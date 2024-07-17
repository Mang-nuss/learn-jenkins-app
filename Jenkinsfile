pipeline {
    agent any

    //for deployment
    environment {
        NETLIFY_SITE_ID = 'fd1215fe-8aca-440f-bad4-22f450f88ecb'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {



    //     stage('Docker') {
    //         steps {
    //             sh 'docker build -t my-playwright .' //build this in the current dir.
    //         }
    //     }
        // stage('w/o docker') {
        //     steps {
        //         sh '''
        //             echo "Without Docker"
        //             ls -latr
        //             touch container-nope.txt
        //         '''
        //     }
        // }
        
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
                    ls -la
                '''
                
/*
                sh '''
                    npx playwright install
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test
                '''
                */
                //The server installaton can be done without the -g (global) tag
            }
        }
        
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }

            environment {
                AWS_S3_BUCKET = 'learn-jenkins-202407161303'
            }

            steps { //trying out the aws CLI commands

                withCredentials([usernamePassword(credentialsId: 'my_jenkins_aws_id', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    // some block
                    sh '''
                        aws --version
                        aws s3 sync build s3://$AWS_S3_BUCKET
                    '''
                }
                //before: 
                        //echo "Hello for S3" > index.html
                        //aws s3 cp index.html s3://$AWS_S3_BUCKET/index.html
            }
        }

        stage('Run Tests') {
            parallel {

                stage('unit tests') {
                
                    agent {
                        docker {
                            image 'node:18-alpine' //was: node:18-alpine  
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
                    netlify --version
                    echo "deploying to STAGING site with ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > staging_output.json
                    node-jq -r '.deploy_url' staging_output.json
                '''
                script {
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' staging_output.json", returnStdout: true)
                }
            }
        }

        stage('Staging E2E test') {
            agent {
                docker {
                    image 'my_playwright' //according to Docker build in the beginning of file.
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'TOFIX' //soon should be STAGING URL
            }

            //#npx playwright install #npm install serve
            steps {
                sh '''
                    serve -s build &
                    sleep 10
                    npx playwright test
                '''
            }
            post {
                always {
                    echo 'post section of staging E2E'
                }
            }
        } 
        
        stage('Approval') {
            steps {
                timeout(time: 15, unit: 'SECONDS') {
                    input message: "Do you want to deploy to prod?", ok: "Yep, I do!"
                    // some block
                }
            }
        }

        // stage('Deploy prod') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'   
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             npm install netlify-cli
        //             node_modules/.bin/netlify --version
        //             echo "deploying to PRODUCTION site with ID: $NETLIFY_SITE_ID"
        //             node_modules/.bin/netlify status
        //             node_modules/.bin/netlify deploy --dir=build --prod
        //         '''
        //     }
        // }
        
        stage('Deploy & Post Prod E2E test; included Deploy prod stage from above') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'TOFIX'
            }
            steps {
                sh '''
                    node --version
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "deploying to PRODUCTION site with ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright test
                '''
            }
            post {
                always {
                    echo 'post section for prod E2E'
                    //junit 'test-results/junit.xml'
                }
            }
        } 
    }    
}