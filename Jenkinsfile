pipeline {
    agent none

    environment {
        NETLIFY_SITE_ID = 'cdf8af0b-86bc-4179-b312-988e98013fa2'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('AWS') {
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    // aws-cli 이미지는 기본적으로 실행 후 바로 종료되므로 엔트리포인트 무력화
                    args "--entrypoint=''" 
                }
            }
            steps {
                sh '''
                    aws --version
                    aws s3 ls
                '''
            }
        }


        stage('Build') {

            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }
            

            steps {
                sh '''
                    echo '트리거 테스트 중 ...'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Test'){

            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }

            steps {
                echo 'Test stage'
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }
        }

        stage('E2E'){

            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }

            steps {
                sh '''
                   npm install serve
                   node_modules/.bin/serve -s build & sleep 10
                   npx playwright test --reporter=html
                '''
            }
        }


        stage('Deploy staging'){
            agent {
                docker { image 'node:18-bullseye' } 
            }

            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "프로젝트 스테이징 배포중.. 사이트 아이디 : $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

        stage('Approval'){
            agent none
            steps{
                timeout(time: 1, unit: 'MINUTES') {
                    input message: '운영 환경에 배포할까요?', ok: '네 배포합니다.'
                }


                
            }
        }




        stage('Deploy prod'){
            agent {
                docker { image 'node:18-bullseye' } 
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "프로젝트 배포중.. 사이트 아이디 : $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E'){
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://poetic-frangipane-7ffc6c.netlify.app'
            }

            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
        }
    }
}
