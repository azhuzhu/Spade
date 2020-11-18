try { // massive try{} catch{} around the entire build for failure notifications
    timestamps {
        timeout(time: 60, unit: 'MINUTES') {
            node('fedora-29') {
                cleanWs()

                stage('Prepare ENV') {
                    sh '''#!/bin/bash
                        git clone https://github.com/release-engineering/Spade.git
                    '''
                } // prepare env stage

                stage('Run Spade'){
                    sh '''#!/bin/bash
                    if [ -n "$MESSAGE_ID" ] ; then
                        export CI_MESSAGE=$(curl "https://datagrepper.engineering.redhat.com/id?id=$MESSAGE_ID")
                    fi
                    cd Spade
                    python3 spade.py -v
                    '''
                } // run spade
            } // node
        } // timeout
    } // timestamps
} catch (e) {
    if (ownership.job.ownershipEnabled) {
        mail to: ownership.job.primaryOwnerEmail,
             cc: ownership.job.secondaryOwnerEmails.join(', '),
             subject: "Jenkins job ${env.JOB_NAME} #${env.BUILD_NUMBER} failed",
             body: "${env.BUILD_URL}\n\n${e}"
    }
    throw e
} finally {
    def currentResult = currentBuild.result ?: 'SUCCESS'
    def script_regex = '${BUILD_LOG_REGEX, regex="^The module", maxMatches=5, showTruncatedLines=false, escapeHtml=true}'
    if (currentResult == 'SUCCESS' && manager.logContains('.* module .*')){
        emailext to: "jwboyer@redhat.com, mnewsome@redhat.com, tstellar@redhat.com, yzhu@redhat.com, tkopecek@redhat.com",
                 subject: "[exd-sp-rhelbld-spade] Module dependency overlap",
                 body: script_regex
    }
}
