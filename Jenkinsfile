#!groovy

import groovy.json.JsonSlurperClassic


node {

    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY_DEV
    def SF_USERNAME=env.SF_USERNAME_DEV
    def SERVER_KEY_CREDENTALS_ID=env.SERVER_KEY_CREDENTALS_ID
    def TEST_LEVEL='RunLocalTests'
    def PACKAGE_NAME='JenkinsfileRepo'
    def PACKAGE_VERSION
    def SF_INSTANCE_URL = env.SF_INSTANCE_URL ?: "https://login.salesforce.com"
    
    def toolbelt = tool 'toolbelt'

    properties([pipelineTriggers([cron('0 11 * * *')])])
    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
    }

    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------
    
    withEnv(["HOME=${env.WORKSPACE}"]) {
        
        withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'server_key_file')]) {

            // -------------------------------------------------------------------------
            // Authorize the Dev Hub org with JWT key and give it an alias.
            // -------------------------------------------------------------------------

            stage('Login em UAT') {
                rc = command "\"${toolbelt}\" force:auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile ${server_key_file} --setdefaultusername --setalias ${SF_ORG_ALIAS}"
                //force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
                if (rc != 0) {
                    error 'Salesforce org authorization failed.'
                }
            }

            
            // -------------------------------------------------------------------------
            // Run unit tests in test sandbox.
            // -------------------------------------------------------------------------
            stage('Rodando testes em UAT') {
                rc = command "\"${toolbelt}\" force:apex:test:run --targetusername ${SF_ORG_ALIAS} --wait 10 --resultformat tap --codecoverage --testlevel ${TEST_LEVEL}"
                if (rc != 0) {
                    error 'Salesforce unit test run in UAT failed.'
                }
            }                      
        }
    }
}

def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
        return bat(returnStatus: true, script: script);
    }   
}  
