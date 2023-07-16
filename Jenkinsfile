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
  }
}