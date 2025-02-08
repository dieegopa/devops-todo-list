pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                git branch: 'master', url: 'https://github.com/dieegopa/devops-todo-list'
            }
        }
        stage('Deploy') {
            steps {
                sh'''
                sam build
                sam deploy --region us-east-1 --stack-name todo-list-aws --capabilities CAPABILITY_IAM --parameter-overrides Stage=production --s3-bucket deploy-todo-prod --no-confirm-changeset --no-fail-on-empty-changeset
                '''
            }
        }
        stage('Rest Test') {
            steps {
                catchError(buildResult: 'ABORTED', stageResult: 'FAILURE') {
                    sh'pytest --junitxml=result-rest.xml test/integration/todoApiTest.py -m prod'
                    junit 'result-rest.xml'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}