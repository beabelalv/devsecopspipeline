pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: python
                    image: python:3
                    command:
                    - cat
                    tty: true
                '''
        }
    }  
    
    stages {

        stage('Run Python Version') {
            steps {
                container('python') {
                    sh 'python3 --version || echo Python 3 is not installed'
                    echo 'Checking Pip...'
                    sh 'pip --version || echo Pip is not installed'
                }
            }
        }

        stage("build") {
            steps {
                sh """
                pip install --user virtualenv
                python3 -m virtualenv env
                . env/bin/activate
                pip install -r requirements.txt
                python3 manage.py check
                """
            }
        }

        stage("test") {
            steps {
                sh """
                pip install --user virtualenv
                python3 -m virtualenv env
                . env/bin/activate
                pip install -r requirements.txt
                python3 manage.py test taskManager
                """
            }
        }

        stage('SonarQube') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner1'
                    withSonarQubeEnv {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        

    }

}
