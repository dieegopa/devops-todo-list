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
    }

    post {
        always {
            cleanWs()
        }
    }
}