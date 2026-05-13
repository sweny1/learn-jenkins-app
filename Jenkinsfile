pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'e25ac398-971e-47e0-ada1-b528e9d087b7'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Cleanup Workspace') {
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
        }
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
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true                  
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            npx serve -s build -l 3000 &
                            sleep 10
                            npx playwright test --reporter=html --output=test-results/playwright-junit.xml
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
      
        }   

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {               
                sh '''
                    echo "Deploying to production environment..."
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version                
                    echo $NETLIFY_SITE_ID
                    echo $NETLIFY_AUTH_TOKEN
                    node_modules/.bin/netlify status
                '''
            }
        }
    }
}