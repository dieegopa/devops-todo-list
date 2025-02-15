pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/dieegopa/devops-todo-list'
                sh 'wget -O samconfig.toml "https://raw.githubusercontent.com/dieegopa/devops-todo-list-config/staging/samconfig.toml?t=$(date +%s)"'
            }
        }
        stage('Static Test') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'SUCCESS') {
                    sh '''
                    flake8 --format=pylint --exit-zero src > flake8.out
                    bandit --exit-zero -r src -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                    '''
                    recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[integerThreshold: 8, threshold: 8.0, type: 'TOTAL'], [criticality: 'ERROR', integerThreshold: 10, threshold: 10.0, type: 'TOTAL']]
                    recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[integerThreshold: 2, threshold: 2.0, type: 'TOTAL'], [criticality: 'ERROR', integerThreshold: 4, threshold: 4.0, type: 'TOTAL']]
                }
            }
        }
        stage('Deploy') {
            steps {
                sh'''
                sam build
                sam deploy --no-confirm-changeset --no-fail-on-empty-changeset
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

    post {
        always {
            cleanWs()
        }
    }
}