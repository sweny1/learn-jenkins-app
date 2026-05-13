pipeline {
    agent any
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

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Starting Test Stage after Build..."

                    if test -f build/index.html; then
                        echo "build/index.html exists."
                    else
                        echo "build/index.html does not exist."
                        exit 1
                    fi

                    npm test
                '''
            }
        }
         stage('E2E Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    args '-u root:root'
                }
            }
            steps {
                sh '''
                    npm install serve
                    npx serve -s build -l 3000 &
                    sleep 10
                    npx playwright test --reporter=junit --output=test-results/playwright-junit.xml

                '''

            }
        }
    }

    post {
        always {
            junit 'test-results/**/*.xml'
        }
    }
}
