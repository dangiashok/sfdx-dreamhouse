#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG='ashok.sdangisfdcdx@gmail.com'
    def SFDC_HOST = 'http://login.salesforce.com'
    def JWT_KEY_CRED_ID = '9c83332b-56c6-49cc-ba86-56902421e9cb'
    def CONNECTED_APP_CONSUMER_KEY='3MVG9d8..z.hDcPIxKIvOh32XDGVdKXJHeRKDhvBeySIgdG5oNRWr6D2aeWqKkFDpr3nO0ab5k7OsAJyEYwY4'
    println 'KEY IS' 
	println HUB_ORG
	println SFDC_HOST
	println CONNECTED_APP_CONSUMER_KEY
    println JWT_KEY_CRED_ID
    def toolbelt = tool 'toolbelt'
	println toolbelt
    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }
	println HUB_ORG
    withCredentials([file(credentialsId: '9c83332b-56c6-49cc-ba86-56902421e9cb', variable: 'jwt_key_file')]) {
        stage('Create Scratch Org') {
            if (isUnix()) {
                rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }else{
                 rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
            }
            if (rc != 0) { error 'hub org authorization failed' }

            // need to pull out assigned username
              if (isUnix()) {
                rmsg = sh returnStdout: true, script: "${toolbelt} force:org:create --definitionfile config/enterprise-scratch-def.json --json --setdefaultusername"
              }else{
                   rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername"
              }
            printf rmsg
            println('Hello from a Job DSL script!')
            println(rmsg)
            def beginIndex = rmsg.indexOf('{')
            def endIndex = rmsg.indexOf('}')
            println(beginIndex)
            println(endIndex)
            def jsobSubstring = rmsg.substring(beginIndex)
            println(jsobSubstring)
            
            def jsonSlurper = new JsonSlurperClassic()
            def robj = jsonSlurper.parseText(jsobSubstring)
            //if (robj.status != "ok") { error 'org creation failed: ' + robj.message }
            SFDC_USERNAME=robj.result.username
            robj = null
            
        }
        
          stage('Push To Test Org') {
              if (isUnix()) {
                    rc = sh returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
              }else{
                  rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
              }
            if (rc != 0) {
                error 'push failed'
            }
            
          }
             
    }
}
