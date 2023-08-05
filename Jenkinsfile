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
                  - name: docker
                    image: docker:dind
                    securityContext:
                      privileged: true
                    tty: true
                '''
        }
    }  
    
    stages {

        stage('PREREQUIREMENTS') {
            steps {
                container('python') {
                    echo 'Cloning reports library'
                    git branch: 'main',
                    url: 'https://github.com/beabelalv/devsecopslibrary.git'

                    sh 'python3 --version || echo Python 3 is not installed'
                    echo 'Checking Pip...'
                    sh 'pip --version || echo Pip is not installed'
                }
            }
        }

        stage("[BUILD]") {
            steps {
                container('python') {
                    sh """
                    pip install --user virtualenv
                    python3 -m virtualenv env
                    . env/bin/activate
                    pip install -r requirements.txt 
                    """
                    //Lo borro pero iria arriba python3 app.py
                }
            }
        }
        
        stage('[TEST]'){
            steps{
                echo '[TEST]'
            }
        }

        stage("SCA: Safety") {
            steps {
                container('docker') {
                    sh 'docker run -v "$(pwd)":/src --rm hysnsec/safety check -r requirements.txt --json | tee oast-results.json'
                }
            }
        }

        stage('SCA: SonarQube') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner1'
                    withSonarQubeEnv {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage("SAST: Trufflehog") {
            steps {
                container('docker') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                        git branch: 'main',
                        url: 'https://github.com/beabelalv/devsecopspipeline.git'
                        sh 'docker run -v "$(pwd)":/src --rm hysnsec/trufflehog file:///src --json | tee trufflehog-results.json'
                    }
                }
            }
        }

        stage("SAST: Bandit") {
            steps {
                container('docker') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                        sh ' docker run -v "$(pwd)":/src --rm hysnsec/bandit -r /src -f json -o /src/bandit-results.json'
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'bandit-results.json', fingerprint: true
                }
            }
        }

        stage("Report Generation: Bandit") {
            steps {
                container('python') {
                    sh """
                    . env/bin/activate
                    python devsecopslibrary/bandit/html_generator.py bandit-results.json
                    """
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'bandit-report.html', fingerprint: true
                }
            }
        }

        stage('[Release]'){
            steps{
                echo '[Release]'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: '**/*results.json', fingerprint: true
            }
        }
        

    }

}
