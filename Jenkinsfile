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
                sam deploy --region us-east-1 --stack-name todo-list-aws --capabilities CAPABILITY_IAM --parameter-overrides Stage=production --s3-bucket deploy-todo --no-confirm-changeset --no-fail-on-empty-changeset
                '''
            }
        }
        stage('Rest Test') {
            steps {
                catchError(buildResult: 'ABORTED', stageResult: 'FAILURE') {
                    sh'pytest --junitxml=result-rest.xml test/integration/todoApiTest.py'
                    junit 'result-rest.xml'
                }
            }
        }
        stage('Promote') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: '8958c5f2-695c-4a0c-b327-de3a47c0ff10', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        git config pull.rebase false
                        git checkout master
                        git pull origin master
                        git merge develop --no-ff -m "Merge develop into master for release"
                        git push git@github.com:dieegopa/devops-todo-list.git master
                    '''
                }
            }
        }
    }
}