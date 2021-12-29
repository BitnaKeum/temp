pipeline {
    agent any

    triggers {
        pollSCM('*/3 * * * *')
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
        AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
        AWS_DEFAULT_REGION = 'us-east-2'
        HOME = '.'
    }

    stages {
        // 레포지토리를 다운
        stage('Prepare') {
            agent any

            steps {
                echo "Lets start Journey!!"

                git url: 'https://github.com/BitnaKeum/temp.git',
                    branch: 'main',
                    credentialsId: 'jenkins'
            }

            post {
                success {
                    echo 'Successfully Pull Repository'
                }

                always {
                    echo 'I tried...'
                }

                cleanup {
                    echo 'after all other post condition'
                }
            }
        }

        // aws s3에 파일을 올림
        stage('Deploy Frontend') {
            steps {
                echo 'Deploying Frontend'

                // 이 폴더로 옮겨가서 파일을 올림
                dir ('./website') {
                    sh '''
                    aws s3 sync ./ s3://billiebucket
                    '''
                }
            }

            post {
                // 작업 성공 시 메일 보냄
                success {
                    echo 'Successfully Deploying Frontend'

                    mail    to: 'beausty23develop@gmail.com',
                            subject: "Deploy Frontend Success",
                            body: "Successfully deployed frontend!"
                }

                // 작업 실패 시 메일 보냄
                failure {
                    echo 'Failed Deploying Frontend'

                    mail    to: 'beausty23develop@gmail.com',
                            subject: "Deploy Frontend Failure",
                            body: "Failure to deploy frontend :()"
                }
            }
        }

        stage('Lint Backend') {
            agent {
                docker {
                    image 'node:latest'
                }
            }

            steps {
                dir ('./server') {
                    sh '''
                    npm install&&
                    npm run lint
                    '''
                }
            }
        }

        stage('Test Backend') {
            agent {
                docker {
                    image 'node:latest'
                }
            }

            steps {
                echo 'Test Backend'

                dir ('./server') {
                    sh '''
                    npm install
                    npm run test
                    '''
                }
            }
        }

        stage('Build Backend') {
            agent any

            steps {
                echo 'Build Backend'

                dir ('.server') {
                    sh '''
                    docker build . -t server --build-arg env=${PROD}
                    '''
                }
            }

            post {
                failure {
                    error 'This pipeline stops here...'
                }
            }
        }

        stage('Deploy Backend') {
            agent any

            steps {
                echo 'Build Backend'

                dir ('./server') {
                    sh '''
                    docker run -p 80:80 -d server
                    '''
                }
            }

            post {
                success {
                    mail    to: 'beausty23develop@gmail.com',
                            subject: "Deploy Success",
                            body: "Successfully deployed!"
                }
            }
        }


    }
}
