pipeline {
    
    agent {
        label 'centos8_01'
    }

    stages {
        stage('Sanitiser') {
            steps {
                deleteDir()
            }
        }
        stage('Validate Build Env') {
            steps {
                sh 'sudo salt-call state.highstate'
            }
        }
        stage('Checkout Source') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: 'master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/christophermarklee/easypost-python.git']]])
            }
        }
        stage('Setup Env') {
            steps {
                sh '''
                    python3.6 -m virtualenv ./venv
                    ./venv/bin/pip install requests six pylint
                    ./venv/bin/pip install -r requirements-tests.txt
                '''
            }
        }
        stage('Tests') {
            steps {
                sh '''
                    ./venv/bin/pylint --exit-zero --output=easypost_pylint.log easypost tests examples
                '''
                recordIssues(tools: [pyLint(pattern: '**/*pylint.log')])
                // Using Test API Key set in Jenkins Credentials manager.
                withCredentials([string(credentialsId: 'EasyPost-Test-API-Key', variable: 'TEST_API_KEY')]) {
                    sh '''
                        export PROD_API_KEY=''
                        export TEST_API_KEY=${TEST_API_KEY}
                        ./venv/bin/py.test --junitxml=test_results.xml -v tests || exit 0
                    '''
                    recordIssues(tools: [junitParser(pattern: '**/test_results.xml')])
                }
            }
        }
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: '**/test_results.xml, **/*pylint.log', fingerprint: true, followSymlinks: false, onlyIfSuccessful: true
            }
        }
    }
    
    post {
        success {
            cleanWs()
        }
    }
}
