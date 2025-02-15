pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                git branch: 'master', url: 'https://github.com/dieegopa/devops-todo-list'
                sh 'wget -O samconfig.toml "https://raw.githubusercontent.com/dieegopa/devops-todo-list-config/production/samconfig.toml?t=$(date +%s)"'
            }
        }
        stage('Deploy') {
            steps {
                sh'''
                sam build
                sam deploy --config-file samconfig.toml --config-env production --no-confirm-changeset --no-fail-on-empty-changeset
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
        stage('Promote') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: '8958c5f2-695c-4a0c-b327-de3a47c0ff10', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        git config pull.rebase false
                        git fetch origin master
                        git checkout -b master origin/master
                        git pull origin master
                        git merge develop --strategy-option=theirs --no-ff -m "Merge develop into master for release"
                        git push git@github.com:dieegopa/devops-todo-list.git master
                    '''
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