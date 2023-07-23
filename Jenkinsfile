pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                    containers:
                    - name: maven
                      image: maven:alpine
                      command:
                      - cat
                      tty: true
                '''
        }
    }
    stages {

        stage('[Plan]'){
            steps{
                echo '[Plan]'
            }
        }

        stage('[Code]'){
            steps{
                echo '[Code]'
            }
        }

        stage('SCM'){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/beabelalv/devsecopspipeline.git'
            }
        }

        stage('[Build]'){
            steps{
                echo '[Build]'
            }
        }

        stage('Run maven') {
            steps {
                container('maven') {
                    sh 'mvn -version'
                }
            }
        }

        stage('[Test]'){
            steps{
                echo '[Test]'
            }
        }

        stage('SCA, Dependency Scan: Dependency Check'){
            steps{
                dependencyCheck additionalArguments: '''-o './'
                    -s './'
                    -f 'ALL' 
                    --prettyPrint
                    --format HTML''', odcInstallation: 'DP-check'
            }
        }

        stage('SCA, Code Quality: SonarQube') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner1'
                    withSonarQubeEnv {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=Petclinic -Dsonar.projectName='Petclinic'"
                    }
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
                // Use the 'archiveArtifacts' step to specify the files to archive
                archiveArtifacts '**/*.xml, **/*.html'
            }
        }

        stage('[Deploy]'){
            steps{
                echo '[Deploy]'
            }
        }

        stage('[Operate]'){
            steps{
                echo '[Operate]'
            }
        }

    }
}