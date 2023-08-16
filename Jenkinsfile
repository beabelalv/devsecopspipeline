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
        
        stage('[TEST]'){
            steps{
                echo '[TEST]'
            }
        }

        stage("SCA: Safety") {
            steps {
                container('docker') {
                    sh '''
                            touch safety-results.json
                            chown 1000:1000 safety-results.json
                            '''

                    sh 'docker run -v "$(pwd)":/src --rm hysnsec/safety check -r requirements.txt --json | tee safety-results.json'
                }
            }
            post {
                always {
                    stash includes: 'safety-results.json', name: 'safety-results'
                }
            }
        }

        stage('SCA: SonarQube') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner1'
                    withSonarQubeEnv {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=VamPi -Dsonar.exclusions=TFM/**/*"
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
                            
                            // Create the file and change its ownership
                            sh '''
                            touch trufflehog-results.json
                            chown 1000:1000 trufflehog-results.json
                            '''
                            
                            sh 'docker run --user 1000:1000 -v "$(pwd)":/src --rm hysnsec/trufflehog file:///src --json | tee trufflehog-results.json'
                        }
                    }
                }
                post {
                    always {
                        stash includes: 'trufflehog-results.json', name: 'trufflehog-results'
                    }
                }
            }

        

        stage("SAST: Bandit") {
            steps {
                container('docker') {
                    catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                        sh 'docker run --user 1000:1000 -v "$(pwd)":/src --rm hysnsec/bandit -r /src --exclude TFM -f json -o /src/bandit-results.json'                    }
                }
            }
            post {
                always {
                    stash includes: 'bandit-results.json', name: 'bandit-results'
                }
            }
        }

        stage('[REPORTS CREATION]'){
            steps{
                echo '[REPORTS CREATION]'
            }
        }
        
        stage('Setup Virtual Environment') {
            steps {
                container('python') {
                    sh 'python3 -m venv venv'
                    sh '. venv/bin/activate'
                    script {
                        // Retrieve requirements.txt from shared library
                        def requirementsContent = libraryResource('requirements.txt')
                        
                        // Write the requirements to a new file in the workspace
                        writeFile file: 'requirements.txt', text: requirementsContent
                        
                        // Continue with your pipeline...
                        sh 'pip install -r requirements.txt'
                    }
                }
            }
        }

        stage("Report Generation: Safety") {
            steps {
                container('python') {
                    script {
                        echo "Activating virtual environment:"
                        sh '. venv/bin/activate'
                        unstash 'safety-results' // Retrieve the stashed Trufflehog results file
                        echo "Workspace directory is: ${env.WORKSPACE}"

                        // Call the generateSafetyReport method with the path to the JSON file
                        generateSafetyReport(json: 'safety-results.json')
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'safety/safety-report.html', fingerprint: true
                }
            }
        }

        stage("Report Generation: Trufflehog") {
            steps {
                container('python') {
                    script {
                        echo "Activating virtual environment:"
                        sh '. venv/bin/activate'
                        unstash 'trufflehog-results' // Retrieve the stashed Trufflehog results file
                        echo "Workspace directory is: ${env.WORKSPACE}"

                        // Call the generateTrufflehogReport method with the path to the JSON file
                        generateTrufflehogReport(json: 'trufflehog-results.json')
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trufflehog/trufflehog-report.html', fingerprint: true
                }
            }
        }

        stage("Report Generation: Bandit") {
            steps {
                container('python') {
                    script {
                        echo "Activating virtual environment:"
                        sh '. venv/bin/activate'
                        sh 'ls -l'
                        unstash 'bandit-results' // This line retrieves the stashed file
                        sh 'ls -l'
                        echo "Workspace directory is: ${env.WORKSPACE}"

                        // Call the generateBanditReport method with the path to the JSON file
                        generateBanditReport(json: 'bandit-results.json')
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'bandit/bandit-report.html', fingerprint: true
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
