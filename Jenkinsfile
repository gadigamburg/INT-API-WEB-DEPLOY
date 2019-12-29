//@Library('Utilities') _
import groovy.json.JsonSlurper
import hudson.model.*
def Current_API_version
def Current_WEB_version
def dev_rep_docker = 'gadigamburg/finalproject'
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
                     dir('INT_DEPLOY') {
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
         stage('Deploy Services on K8S'){
             steps{
                 script{
                     try{
                         withCredentials([usernamePassword(credentialsId: 'docker_hub', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                            println("DockerUser: ${DOCKER_USERNAME}")
                            println("DockerPass: ${DOCKER_PASSWORD}")
                            sh "docker login -u=${DOCKER_USERNAME} -p=${DOCKER_PASSWORD}"
                            sh "docker tag $module$colons$BuildVersion $dev_rep_docker$colons$module$underscore$BuildVersion"
                            sh "docker push $dev_rep_docker$colons$module$underscore$BuildVersion"
                         }
                     }
                     catch (exception){
                         println "Deployment Process Failed!!!"
                         currentBuild.result = 'FAILURE'
                         throw exception
                     }
                 }
             }
         }
    }
}

def Return_Json_From_File(file_name){
    return new JsonSlurper().parse(new File(file_name))
}