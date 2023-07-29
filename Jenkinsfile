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

        stage("oast") {
            steps {
                container('docker') {
                sh "docker run -v \$(pwd):/src --rm hysnsec/safety check -r requirements.txt --json | tee oast-results.json"
                }
            }
        }
        
        stage("integration") {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    echo "This is an integration step."
                    sh "exit 1"
                }
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
