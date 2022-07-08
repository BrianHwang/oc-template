def templateFileLocation = "template"

def name = "HelloWorld"
def namespace = "cp4i-poc"

def repo = "https://github.com/BrianHwang/ace-bar/raw/main"
def barName = "MC_HelloWorld.bar"

def configurationList = "brian-github"


   
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
				sh label: '', script: '''#!/bin/bash
				    echo ${templateFileLocation}
					echo ${name}
					echo ${namespace}
					echo ${repo}
					echo ${barName}
					echo ${configurationList}
					echo ''
                    set -e
                    cd ${templateFileLocation}
					BAR_FILE="${repo}/${barName}"
					cat ace-template.yaml.temp
					sed -e "s//{{NAME}}/${name}/g" \
					    -e "s//{{NAMESPACE}}/${namespace}/g" \
					    -e "s//{{BAR}}/$BAR_FILE/g" \
					    -e "s//{{CONFIGURATION_LIST}}/${configurationList}/g" \
						ace-template.yaml.temp > ace.yaml
                    cat ace.yaml
                    oc apply -f ace.yaml
                    '''
            }
        }
    }
}
