pipeline {
    agent any
    environment{
        NETLIFY_SITE_ID = 'da550e5c-ef55-4f0b-ac9a-98c743350cb4'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    stages {
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }         
            }
            steps {
                sh '''
                echo "Small change"
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
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }         
                    }
                    steps {
                        sh '''
                            echo 'Test stage'
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post{
                        always{
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E') {
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.51.0-noble'
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
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }         
            }
            steps {
                sh '''
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to production Site ID: $NETLIFY_SITE_ID"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('Prod E2E') {
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.51.0-noble'
                            reuseNode true
                        }         
                    }
                    environment{
                    CI_ENVIRONMENT_URL='https://clever-bonbon-f465b9.netlify.app'
                    }
                    steps {
                        sh '''
                            npx playwright test --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
    }
}