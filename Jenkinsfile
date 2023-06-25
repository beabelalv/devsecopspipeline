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

        stage('Run Tests and Generate Coverage Report') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pytest --cov=models --cov-report xml:coverage.xml'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                def scannerHome = tool 'SonarScanner';
                withSonarQubeEnv('Deployed SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
        }
    }
}
}
