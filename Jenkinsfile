pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
				sh label: '', script: '''#!/bin/bash
                    set -e
                    cd template
                    cp ace-template.yaml.temp ace-template.yaml
                    cat ace-template.yaml
                    oc apply -f integration-server.yaml
                    '''
            }
        }
    }
}
