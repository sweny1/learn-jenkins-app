pipeline {
    agent any

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
    }
}