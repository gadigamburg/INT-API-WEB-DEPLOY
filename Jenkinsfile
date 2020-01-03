//@Library('Utilities') _
import groovy.json.JsonSlurper
import hudson.model.*
def Current_API_version
def Current_WEB_version
def dev_rep_docker = 'gadigamburg/finalproject'
def deployment_name = 'api&web'
def colons = ':'
def module = 'intdeploy'
def underscore = '_'

pipeline {
    options {
        timeout(time: 30, unit: 'MINUTES')
    }
    agent { label 'master' }
    stages {
         stage('Checkout') {
             steps {
                 script {
                     dir('Release') {
                         deleteDir()
                         checkout([$class: 'GitSCM', branches: [[name: 'gadi']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'INT_API', url: "https://github.com/gadigamburg/Release.git"]]])
                         println("Test")
                         path_json_file = sh(script: "pwd", returnStdout: true).trim() + '/' + 'release' + '.json'
                         println("path_json_file: $path_json_file")
                         Current_API_version = Return_Json_From_File("$path_json_file").release.services.intapi.version
                         Current_WEB_version = Return_Json_From_File("$path_json_file").release.services.intweb.version
                         println("Current_API_version: $Current_API_version")
                         println("Current_WEB_version: $Current_WEB_version")
                     }
                     dir('INT_API+WEB-CD') {
                         deleteDir()
                         checkout([$class: 'GitSCM', branches: [[name: 'gadi']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'INT_API', url: "https://github.com/gadigamburg/INT-API-WEB-DEPLOY.git"]]])
                     }
                 }
             }
         }
         stage('UT') {
             steps {
                 println('UT will be added soon...')
             }
         }
         stage('Deployment with Kubernetes'){
             steps{
                 script{
                     dir('INT_API+WEB-CD'){
                         sh "sed -i 's/{{API_Version}}/$Current_API_version/' Deploy.yml"
                         sh "sed -i 's/{{WEB_Version}}/$Current_WEB_version/' Deploy.yml"
                         /*
                         sh """
                            export PATH=/bin/bash:$PATH
                            export KUBECONFIG=/var/jenkins_home/admin.conf
                            kubectl apply -f Deploy.yml
                            kubectl patch deployment $deployment_name -p '{"spec":{"progressDeadlineSeconds":15}}'
                            if ! kubectl rollout status deployment $deployment_name; then
                                    kubectl rollout undo deployment $deployment_name
                            fi
                         """
                         */
                     }
                 }
             }
         }
    }
}

def Return_Json_From_File(file_name){
    return new JsonSlurper().parse(new File(file_name))
}