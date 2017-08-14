env.BUILD_TIMESTAMP = new java.text.SimpleDateFormat('yyyyMMddHHmmss').format(new Date())
def credentialsId = 'c1-github-password-for-mobileciuser'
node(env.NODE_LABEL) {
    // Need to include all the parameters used by the shell
    withEnv([
        "BRANCH_NAME=$BRANCH_NAME",
        "SCM_URL=$SCM_URL",
        "HTTP_PROXY=$HTTP_PROXY",
        "HTTPS_PROXY=$HTTPS_PROXY",
        "NO_PROXY=$NO_PROXY"
    ]) {
        withCredentials([
                usernamePassword(credentialsId: 'c1-github-password-for-mobileciuser',
                usernameVariable: 'GH_USER', passwordVariable: 'GH_PW')
        ]) {
            
            // Empty workspace
            deleteDir()

            // Build RCs only on master branch
            git url: env.SCM_URL,
                    credentialsId: "c1-github-password-for-mobileciuser",
                    branch: "${env.BRANCH_NAME}"

            def pom = readMavenPom()
            // Note: getArtifactID and getVersion methods need to be whitelisted in Jenkins master
            env.ANSIBLE_FORCE_COLOR = "true"
            env.ARTIFACT_NAME = pom.getArtifactId()
            env.ARTIFACT_VERSION = pom.getVersion()
            //env.ARTIFACT_VERSION = env.ARTIFACT_VERSION.replaceAll(/[-a-zA-Z]/, "")
            env.ARTIFACT_VERSION = env.ARTIFACT_VERSION.split("-")[0]
            echo "INFO: Building artifact [${env.ARTIFACT_NAME}] version [${env.ARTIFACT_VERSION}]"

            // Assign Maven tool from Global Tools
            env.mvnHome = tool "Maven3.3.9"
            env.MAVEN_OPTS = "-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true -Dmaven.javadoc.failOnError=false"
            echo "INFO: maven [${mvnHome}] MAVEN_OPTS [${MAVEN_OPTS}]"

            wrap([$class: 'AnsiColorBuildWrapper']) {
                sh script: '''#!/bin/bash
                    set -ex
                    git config --local user.email "${GH_USER}@capitalone.com"
                    git config --local user.name "${GH_USER}"
                    git tag -a rc-${BUILD_TIMESTAMP} -m 'rc from jenkins'
                    git config --local credential.username ${GH_USER}
                    echo "https://${GH_USER}:${GH_PW}@github.kdc.capitalone.com" > .git/bollocks
                    git config credential.helper store
                    git config --local credential.helper "store --file=.git/bollocks"
                    git push origin rc-${BUILD_TIMESTAMP}
                    rm .git/bollocks
                    git config --local --remove-section credential
                    git config --local --remove-section user
                '''
            }

        }
    }

}
