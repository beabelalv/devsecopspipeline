pipeline {
    agent {
        docker {
            image 'python:3.9' // Use the official Python Docker image
            args '-v /var/run/docker.sock:/var/run/docker.sock' // To use Docker in Docker
        }
    }

    stages {

        stage('Prerequirements') {
            steps {
                sh 'apt install python3 -y'
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
*/        }

        stage('SonarQube') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner1';
                    withSonarQubeEnv() {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }

                }
        }
    }
}
}
