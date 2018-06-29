pipeline {
    agent none
    stages {
        stage ('Initialize') {
            agent { label 'concierge-jp-workforce' }
            steps {
                sh 'printenv'
            }
        }
        stage ('Sample') {
            agent { label 'concierge-jp-workforce' }
            steps {
                sh 'ls'
                sh "echo $SSH_CONNECTION"
            }
        }
    }
}
