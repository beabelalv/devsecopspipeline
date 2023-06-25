pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello, World!'
            }
        }

        stage('Clone and compile') {
            steps {
                git 'https://github.com/erev0s/VAmPI'
            }
        }
    }
}
