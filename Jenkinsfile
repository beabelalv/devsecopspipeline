pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello, World!'
            }
        }

        stage('SCM') {
            steps {
                checkout scm
            }
        }

        stage('Prerequirements') {
            steps {
                echo 'Checking Python...'
                sh 'python3 --version || echo Python 3 is not installed'
                echo 'Checking Pip...'
                sh 'pip --version || echo Pip is not installed'
                echo 'Printing PATH...'
                sh 'echo $PATH'
            }

        }

/*
        stage('Run Tests and Generate Coverage Report') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pytest --cov=models --cov-report xml:coverage.xml'
            }
        }
*/
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner';
                    withSonarQubeEnv('Deployed SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }

                }
        }
    }
}
}
