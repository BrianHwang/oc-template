def ocImage = "image-registry.openshift-image-registry.svc:5000/cp4i-poc/oc-deploy:latest"

def namespace = "cp4i-poc"
def serverName = "HelloWorld"

def repoPath = "https://github.com/BrianHwang/ace-bar/raw/main"
def barName = "MC_HelloWorld.bar"

def configurationList = "brian-github"
def projectDir = "oc-template"
def gitDomain = "github.com"
def gitRepo = "https://github.com/BrianHwang/oc-template.git"


podTemplate(
    serviceAccount: "jenkins",
    containers: [
       
        containerTemplate(name: 'oc-deploy', image: "${ocImage}", workingDir: "/home/jenkins", ttyEnabled: true, 
          envVars: [
            envVar(key: 'NAMESPACE', value: "${namespace}"),
            envVar(key: 'SERVER_NAME', value: "${serverName}"),
            envVar(key: 'REPO_PATH', value: "${repoPath}"),
            envVar(key: 'BAR_NAME', value: "${barName}"),            
            envVar(key: 'CONFIGURATION_LIST', value: "${configurationList}"),
            envVar(key: 'PROJECT_DIR', value: "${projectDir}"),
        ]),
        containerTemplate(name: 'jnlp', image: "jenkins/jnlp-slave:latest", ttyEnabled: true, workingDir: "/home/jenkins", 
		  envVars: [
            envVar(key: 'HOME', value: '/home/jenkins'),
            envVar(key: 'GIT_DOMAIN', value: "${gitDomain}"),
            envVar(key: 'GIT_REPO', value: "${gitRepo}"),
            envVar(key: 'PROJECT_DIR', value: "${projectDir}"),
        ])		
  ]) {
    node(POD_LABEL) {
        stage('Git Checkout') {
            container("jnlp") {
                sh """
                    echo "**************************************************************"
                    git clone $GIT_REPO
                    cp -p $PROJECT_DIR/template/ace-template.yaml.temp $PROJECT_DIR
                    pwd
                    ls -la
                    cd $PROJECT_DIR
                    pwd
                    ls -la
                    echo "**************************************************************"
                """
            }
        }	
        stage('Deploy Intergration Server') {
            container("oc-deploy") {
                sh label: '', script: '''#!/bin/bash                    
					set -e
					echo "****************************************************************"
                    cd $PROJECT_DIR
                    cat ace-template.yaml.temp
					echo "****************************************************************"
                    sed -e "s/{{NAME}}/$SERVER_NAME/g" \
                        -e "s/{{NAMESPACE}}/$NAMESPACE/g" \
                        -e "s/{{REPO_PATH}}/$REPO_PATH/g" \
                        -e "s/{{BAR}}/$BAR_NAME/g" \
                        -e "s/{{CONFIGURATION_LIST}}/$CONFIGURATION_LIST/g" \
                        ace-template.yaml.temp > ace.yaml
                    
                    cat ace.yaml
                   
					echo "****************************************************************"
					
					oc apply -f ace.yaml
                    '''
            }
        }
    }
}