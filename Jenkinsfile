pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = 'a9728843-2e0d-41dd-9e01-d32769d725c1'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
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
                        test -f build/index.html
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
                         image 'my-playwright'
                         reuseNode true
                       }
                    }
            
                    steps {
                     sh '''
                      serve -s build &
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
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment{  
                CI_ENVIRONMENT_URL = 'This is satge experiement'
            }
                    
            steps {
                sh '''
                    netlify --version
                    echo 'Deploying to site ID: $NETLIFY_SITE_ID'
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                   publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Staging', reportTitles: '', useWrapperFileDirectly: true])
                }  
            }       
        }  

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment{  
                CI_ENVIRONMENT_URL = 'https://musing-ritchie-5b7167.netlify.app'
            }
                    
            steps {
                sh '''
                    node --version
                    netlify --version
                    echo "Deploying to site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
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
