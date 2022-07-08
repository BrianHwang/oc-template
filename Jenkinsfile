pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                sh "oc get po -n cp4i-poc"
            }
        }
    }
}
