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
        stage('SCM'){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/beabelalv/devsecopspipeline.git'
            }
        }

        stage('Run maven') {
            steps {
                container('maven') {
                    sh 'mvn -version'
                }
            }
        }

        stage('Dependency Check'){
            steps{
                dependencyCheck additionalArguments: '--format HTML', odcInstallation: 'DP-check'
            }
        }

        stage('SonarQube') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner1'
                    withSonarQubeEnv {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=Petclinic -Dsonar.projectName='Petclinic'"
                    }
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                // Use the 'archiveArtifacts' step to specify the files to archive
                archiveArtifacts '**/*.xml, **/*.html'
            }
        }
    }
}