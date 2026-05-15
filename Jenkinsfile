pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'e25ac398-971e-47e0-ada1-b528e9d087b7'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
    }


    stages {

        stage('AWS'){

            agent {
                docker {
                    image 'amazon/aws-cli:2.11.7 --entrypoint=""'
                    reuseNode true
                }
            }

            steps{
                sh '''
                    echo "AWS CLI version:"
                    aws --version
                '''
            }
        }
       
       /* stage('Cleanup Workspace') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '-u root:root'
                }
            }
            steps {
                sh '''
                    echo "Cleaning old files with root permission..."
                    rm -rf node_modules build test-results playwright-report
                '''
            }
        } */
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Starting Build Stage..."

                    ls -la
                    node --version
                    npm --version

                    npm ci
                    npm run build

                    echo "Build completed successfully"
                    ls -la
                '''
            }
        }

        stage('Tests'){
            parallel {
                stage('Unit tests') {
                    agent {
                         docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }                                       
                    }
                    steps {
                        sh '''
                            mkdir -p jest-results

                            JEST_JUNIT_OUTPUT_DIR=jest-results \
                            JEST_JUNIT_OUTPUT_NAME=junit.xml \
                            CI=true npm test -- --watchAll=false
                        '''
                    }
                    post {
                        always {
                            junit allowEmptyResults: true, testResults: 'jest-results/junit.xml'
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
                            serve -s build -l 3000 &
                            sleep 10
                            npx playwright test --reporter=html --output=test-results/playwright-junit.xml
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'E2E local', reportTitles: '', useWrapperFileDirectly: true])
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

          

            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }

            steps {
                echo "Staging E2E is starting at ${env.CI_ENVIRONMENT_URL}"
                sh '''                 
                    netlify --version
                    echo "Checking build directory..."                
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                      publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'E2E staging',
                        ])
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

            environment {
                CI_ENVIRONMENT_URL = 'https://lucky-scone-143ddb.netlify.app'
            }

            steps {
                sh '''
                    node --version
                    echo "Deploying to production environment..."
                    npm install netlify-cli@20.1.1
                    netlify --version                
                    echo $NETLIFY_SITE_ID
                    echo $NETLIFY_AUTH_TOKEN
                    netlify status
                    netlify deploy --dir=build --prod 
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                      publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'E2E Prod',
                        ])
                }
            }
        }

    }
}