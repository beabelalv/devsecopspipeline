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

    stage('Dependency-Check Analysis') {
        steps {
            script {
                dependencyCheckAnalyzer datadir: 'dependency-check-data', hintsFile: '', includeCsvReports: false, includeHtmlReports: true, includeJsonReports: false, includeVulnReports: true, isAutoupdate: true, outdir: '', scanpath: '', skipOnScmChange: false, skipOnUpstreamChange: false, suppressionFile: '', zipExtensions: ''
            }
        }
    }

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

  post {
      always {
          dependencyCheckPublisher pattern: '**/dependency-check-report.xml', failedTotalHigh: '0', failedTotalNormal: '0', failedTotalLow: '0'
      }
  }

}