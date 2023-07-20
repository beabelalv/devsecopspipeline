pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: python
            image: beatrizbeltranalv/python-image:latest
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

    stage ('Source Composition Analysis: DependencyCheck') {
      steps {
         container('python') {
          sh 'rm owasp* || true'
          sh 'wget "https://github.com/beabelalv/devsecopspipeline/blob/main/owasp-dependency-check.sh" '
          sh 'chmod +x owasp-scan.sh'
          sh 'bash owasp-dependency-check.sh'
          sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
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
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'      
      }
  }


}