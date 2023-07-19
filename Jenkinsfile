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

    stage('Dependency-Check') {
                steps {
                    script {
                        dependencyCheck additionalArguments: '--scan target/*.jar'
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
      dependencyCheckPublisher canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '', unHealthy: ''
    }
  }
}