@Library('report-generator') _

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
                    sh 'python3 --version || echo Python 3 is not installed'
                    echo 'Checking Pip...'
                    sh 'pip --version || echo Pip is not installed'

                    sh 'pwd' // prints current working directory
                    sh 'ls -R' // prints directory structure
                }
            }
        }

        // stage("[BUILD]") {
        //     steps {
        //         container('python') {
        //             sh """
        //             pip install --user virtualenv
        //             python3 -m virtualenv env
        //             . env/bin/activate
        //             pip install -r requirements.txt 
        //             """
        //             //Lo borro pero iria arriba python3 app.py
        //         }
        //     }
        // }
        
        // stage('[TEST]'){
        //     steps{
        //         echo '[TEST]'
        //     }
        // }

        // stage("SCA: Safety") {
        //     steps {
        //         container('docker') {
        //             sh 'docker run -v "$(pwd)":/src --rm hysnsec/safety check -r requirements.txt --json | tee oast-results.json'
        //         }
        //     }
        // }

        // stage('SCA: SonarQube') {
        //     steps {
        //         script {
        //             def scannerHome = tool 'SonarScanner1'
        //             withSonarQubeEnv {
        //                 sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=VamPi -Dsonar.exclusions=TFM/**/*"
        //             }
        //         }
        //     }
        // }

        // stage("SAST: Trufflehog") {
        //     steps {
        //         container('docker') {
        //             catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
        //                 git branch: 'main',
        //                 url: 'https://github.com/beabelalv/devsecopspipeline.git'
        //                 sh 'docker run -v "$(pwd)":/src --rm hysnsec/trufflehog file:///src --json | tee trufflehog-results.json'
        //             }
        //         }
        //     }
        // }
        

        stage("SAST: Bandit") {
            steps {
                container('docker') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                        sh ' docker run -v "$(pwd)":/src --rm hysnsec/bandit -r /src --exclude TFM -f json -o /src/bandit-results.json'                    
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'bandit-results.json', fingerprint: true
                }
            }
        }
        
        stage('Setup Virtual Environment') {
            steps {
                container('python') {
                    sh 'python3 -m venv venv'
                    sh 'source venv/bin/activate'

                    // Retrieve requirements.txt from shared library and copy to current workspace
                    def requirementsPath = libraryResource('resources/requirements.txt')
                    sh "cp ${requirementsPath} ."

                    // Install packages from requirements.txt
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage("Report Generation: Bandit") {
            steps {
                container('python') {
                    sh """
                    echo "Activating virtual environment:"
                    . env/bin/activate
                    echo "Current working directory:"
                    pwd
                    echo "Listing bandit directory:"
                    ls -l bandit
                    echo "Trying to run the script:"
                    python bandit/html_generator.py reports/bandit-results.json
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
