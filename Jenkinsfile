pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = 'a9728843-2e0d-41dd-9e01-d32769d725c1'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
            stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'Small changess'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        
        stage('Tests'){
            parallel{
                
               stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                     sh '''
                        #test -f build/index.html
                        npm test
                      '''
                    }

                    post{
                       always {
                         junit 'jest-results/junit.xml'
                       }
                    }
                }            


                stage('E2E Test') {
                   agent {
                       docker {
                         image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                         reuseNode true
                       }
                    }
            
                    steps {
                     sh '''
                      npm install serve
                      node_modules/.bin/serve -s build &
                      sleep 10
                      npx playwright test --reporter=html

                      '''
                    }

                    post {
                      always {
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Build', reportTitles: '', useWrapperFileDirectly: true])
                       }  
                    }       
                }
        
            }

        }    

        stage('Deploy Staging') {
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
                    echo "Deploying to site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build 

                '''
            }
        }

        stage('Approved') {
            steps {
                
                timeout(1) {
                 input message: 'Ready to Deploy?', ok: 'Stimmt ! Bitte Deploy...'
                }
            }   
        }

        stage('Deploy Prod') {
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
                    echo "Deploying to site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod

                '''
            }
        }

        stage('E2E Prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment{  
                CI_ENVIRONMENT_URL = 'https://musing-ritchie-5b7167.netlify.app'
            }
                    
            steps {
                sh '''
                npm install serve
                node_modules/.bin/serve -s build &
                sleep 10
                npx playwright test --reporter=html
                '''
            }

            post {
                always {
                   publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Prod', reportTitles: '', useWrapperFileDirectly: true])
                }  
            }       
        }   

    }       
 
}
