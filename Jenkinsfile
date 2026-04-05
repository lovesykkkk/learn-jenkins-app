pipeline {
    agent none

    environment {
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        AWS_ECS_CLUSTER = 'hollow-rabbit-1mib6o'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod3'
    }


    stages {

        stage('Build Docker image') {
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint='' -v /var/run/docker.sock:/var/run/docker.sock"
                }
            }

            steps {

                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        yum install -y docker
                        docker build --platform linux/amd64 -t myjenkinsapp .
                    '''
                }

                
            }
        }

        


        stage('Build') {

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy' 
                    reuseNode true
                }
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

        

        
    }
}
