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
   parameters {
        booleanParam(defaultValue: true, description: 'Enable/Disable PREREQUIREMENTS stage', name: 'PREREQUIREMENTS_ENABLED')
        booleanParam(defaultValue: false, description: 'Enable/Disable [BUILD] stage', name: 'BUILD_ENABLED')
        booleanParam(defaultValue: false, description: 'Enable/Disable [TEST] stage', name: 'TEST_ENABLED')
        booleanParam(defaultValue: false, description: 'Enable/Disable SCA: Safety stage', name: 'SCA_Safety_ENABLED')
        booleanParam(defaultValue: false, description: 'Enable/Disable SCA: SonarQube stage', name: 'SCA_SonarQube_ENABLED')
        booleanParam(defaultValue: false, description: 'Enable/Disable SAST: Trufflehog stage', name: 'SAST_Trufflehog_ENABLED')
        booleanParam(defaultValue: true, description: 'Enable/Disable SAST: Bandit stage', name: 'SAST_Bandit_ENABLED')
        booleanParam(defaultValue: true, description: 'Enable/Disable Report Generation: Bandit stage', name: 'Report_Generation_Bandit_ENABLED')
        booleanParam(defaultValue: true, description: 'Enable/Disable [Release] stage', name: 'Release_ENABLED')
        booleanParam(defaultValue: true, description: 'Enable/Disable Archive Artifacts stage', name: 'Archive_Artifacts_ENABLED')
    }
 }  
    
    stages {

        stage('PREREQUIREMENTS') {
            when { expression { params.PREREQUIREMENTS_ENABLED } } {
            steps {
                container('python') {
                    echo 'Cloning reports library'
                    git branch: 'main',
                    url: 'https://github.com/beabelalv/devsecopslibrary.git'

                    sh 'python3 --version || echo Python 3 is not installed'
                    echo 'Checking Pip...'
                    sh 'pip --version || echo Pip is not installed'

                    sh 'pwd' // prints current working directory
                    sh 'ls -R' // prints directory structure
                }
            }
        }

        stage('[BUILD]') {
            when { expression { params.BUILD_ENABLED } } {
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
        
        stage('[TEST]') {
            when { expression { params.TEST_ENABLED } }{
            steps{
                echo '[TEST]'
            }
        }

        stage('SCA: Safety') {
            when { expression { params.SCA_Safety_ENABLED } } {
            steps {
                container('docker') {
                    sh 'docker run -v "$(pwd)":/src --rm hysnsec/safety check -r requirements.txt --json | tee oast-results.json'
                }
            }
        }

        stage('SCA: SonarQube') {
            when { expression { params.SCA_SonarQube_ENABLED } } {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner1'
                    withSonarQubeEnv {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=VamPi -Dsonar.exclusions=**/devsecopslibrary/TFM/**/*"
                    }
                }
            }
        }

        stage('SAST: Trufflehog') {
            when { expression { params.SAST_Trufflehog_ENABLED } } {
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
        

        stage('SAST: Bandit') {
            when { expression { params.SAST_Bandit_ENABLED } } {
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
        

        stage('Report Generation: Bandit') {
            when { expression { params.Report_Generation_Bandit_ENABLED } } {
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

        stage('[Release]') {
            when { expression { params.Release_ENABLED } }{
            steps{
                echo '[Release]'
            }
        }

        stage('Archive Artifacts') {
            when { expression { params.Archive_Artifacts_ENABLED } } {
            steps {
                archiveArtifacts artifacts: '**/*results.json', fingerprint: true
            }
        }
        

    }

}
