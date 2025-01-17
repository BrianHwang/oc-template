def ocImage =         "image-registry.openshift-image-registry.svc:5000/cp4i-poc/oc-deploy:latest"
def barBuilderImage = "image-registry.openshift-image-registry.svc:5000/cp4i-poc/ace-bar-builder:12.0.4.0-ubuntu"

def gitRepo = "https://github.com/BrianHwang/oc-template.git"
def aceSourceCodeRepo = "https://github.com/BrianHwang/MC_HelloWorld.git"
def barRepo = "https://github.com/BrianHwang/ace-bar.git"

def namespace = "cp4i-poc"
def serverName = "helloworld"

def appName = "MC_HelloWorld"
def barName = "MC_HelloWorld.bar"

def configurationList = "brian-github"
def projectDir = "oc-template"
def sourcecodeDir = "MC_HelloWorld"

def gitDomain = "github.com"



podTemplate(
    serviceAccount: "jenkins",
    containers: [
        containerTemplate(name: 'jnlp', image: "jenkins/jnlp-slave:latest", ttyEnabled: true, workingDir: "/home/jenkins", 
		  envVars: [
            envVar(key: 'GIT_REPO', value: "${gitRepo}"),
            envVar(key: 'GIT_ACE_REPO', value: "${aceSourceCodeRepo}"),
            envVar(key: 'PROJECT_DIR', value: "${projectDir}"),
            envVar(key: 'BAR_REPO', value: "${barRepo}"),
            envVar(key: 'APP_NAME', value: "${appName}"),

        ]),
        containerTemplate(name: 'oc-deploy', image: "${ocImage}", workingDir: "/home/jenkins", ttyEnabled: true, 
          envVars: [
            envVar(key: 'NAMESPACE', value: "${namespace}"),
            envVar(key: 'SERVER_NAME', value: "${serverName}-${BUILD_NUMBER}"),
            envVar(key: 'BAR_NAME', value: "${barName}"),            
            envVar(key: 'CONFIGURATION_LIST', value: "${configurationList}"),
            envVar(key: 'PROJECT_DIR', value: "${projectDir}"),
            envVar(key: 'SOURCE_CODE_DIR', value: "${sourcecodeDir}"),
            envVar(key: 'APP_NAME', value: "${appName}"),
        ]),
        containerTemplate(name: 'ace-bar-builder', image: "${barBuilderImage}", workingDir: "/home/jenkins", ttyEnabled: true, 
          envVars: [
            envVar(key: 'BAR_NAME', value: "${barName}"),
            envVar(key: 'APP_NAME', value: "${appName}"),
            envVar(key: 'PROJECT_DIR', value: "${projectDir}"),
        ])	
  ]) {
    node(POD_LABEL) {
        stage('Git Checkout') {
            container("jnlp") {
                sh """
                    echo "**************************************************************"
                    echo "     Git Checkout"
                    echo "**************************************************************"
                    git clone $GIT_REPO
                    git clone $GIT_ACE_REPO
                    git clone $BAR_REPO
                    cp -p $PROJECT_DIR/template/ace-template.yaml.temp $PROJECT_DIR
                    pwd
                    ls -la
                    cd $PROJECT_DIR
                    pwd
                    ls -la
                """
            }
        }
        stage('Build Bar File') {
            container("ace-bar-builder") {
                sh label: '', script: '''#!/bin/bash
                    echo "**************************************************************"
                    echo "     Build Bar File"
                    echo "**************************************************************"
                    Xvfb -ac :99 &
                    export DISPLAY=:99
                    export LICENSE=accept
                    pwd
                    source /opt/ibm/ace-12/server/bin/mqsiprofile
                    cd /home/jenkins/workspace/ace-build/$SOURCE_CODE_DIR
                    pwd
                    BAR_FILE="${APP_NAME}_${BUILD_NUMBER}.bar"
                    mqsicreatebar -data . -b $BAR_FILE -a $APP_NAME -cleanBuild -trace -configuration . 
                    ls -lha
                    '''
            }
        }
        stage('Upload Bar File') {
            container("jnlp") {
                
                sh label: '', script: '''#!/bin/bash
                    echo "**************************************************************"
                    echo "     Upload Bar File"
                    echo "**************************************************************"
                    BAR_FILE="${APP_NAME}_${BUILD_NUMBER}.bar"
                    
                    set -e 
                    cd /home/jenkins/workspace/ace-build
                    pwd
                    ls -lha

                    cp -p $BAR_FILE ace-bar
                    cd ace-bar
                    pwd
                    ls -lha
                        
                    git config --global user.email "brian_hwang@itss.vic.gov.au"
                    git config user.name brian_hwang
                        
                    git add $BAR_FILE
                    git status
                        
                    git commit -m "jenkin build bar file"
                        
                '''
                
                withCredentials([usernamePassword(credentialsId: 'brian_github_credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    script {
                            env.encodedPass=URLEncoder.encode(PASS, "UTF-8")
                            env.USER_NAME=USER
                    }
                }

                sh label: '', script: '''#!/bin/bash
                    
                    pwd
                    cd /home/jenkins/workspace/ace-build/ace-bar
                    pwd
                    git push https://${USER_NAME}:${encodedPass}@github.com/BrianHwang/ace-bar.git
                 '''

            }
        }	
        stage('Deploy Intergration Server') {
            container("oc-deploy") {
                sh label: '', script: '''#!/bin/bash
                    echo "**************************************************************"
                    echo "     Deploy Intergration Server"
                    echo "**************************************************************"                
					
                    BAR_FILE="${APP_NAME}_${BUILD_NUMBER}.bar"
                    
                    set -e
                    pwd
                    cd $PROJECT_DIR
                    echo $BAR_FILE
                    cat ace-template.yaml.temp
					
                    sed -e "s/{{NAME}}/$SERVER_NAME/g" \
                        -e "s/{{NAMESPACE}}/$NAMESPACE/g" \
                        -e "s/{{BAR_NAME}}/$BAR_FILE/g" \
                        -e "s/{{CONFIGURATION_LIST}}/$CONFIGURATION_LIST/g" \
                        ace-template.yaml.temp > ace.yaml
                    
                    cat ace.yaml
                   
					oc apply -f ace.yaml
                '''
            }
        }
    }
}