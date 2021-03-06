// Powered by Infostretch

pipeline {
    agent any

    options {
        ansiColor('xterm')
    }
    stages {
        stage('Notify Start') {
            steps {
                withCredentials([string(credentialsId: 'teams-webhook-url', variable: 'WEBHOOK')]) {
                    script {
                        msg = sh(script: "git log -1 --oneline", returnStdout: true)
                        slackSend(
                                channel: 'nimbus',
                                message: "${env.JOB_NAME} - ${currentBuild.displayName} ${currentBuild.buildCauses[0].shortDescription} (<${env.JOB_URL}|Open>)",
                                color: (currentBuild.previousBuild?.result == 'SUCCESS') ? 'good' : 'danger'
                        )
                        office365ConnectorSend(
                                color:  (currentBuild.previousBuild?.result == 'SUCCESS') ? '00FF00' : 'FF0000',
                                message: "Build ${currentBuild.displayName} triggered by ${currentBuild.buildCauses[0].shortDescription}\r\n\r\n${msg}",
                                webhookUrl: "${env.WEBHOOK}",
                                status: "Building"
                        )
                    }
                }
            }
        }
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '**']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'a345baf7-5ec4-404c-a309-413b45b31f48', url: 'https://github.com/admpresales/nimbus-dockerapp']]])
            }
        }
        stage('Build') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-nimbusbuildserver', passwordVariable: 'HUB_PASS', usernameVariable: 'HUB_USER')]) {
                    sh """ 
                        echo \$GIT_PREVIOUS_COMMIT
                        echo \$GIT_COMMIT
                        CHANGED=\$(git diff --diff-filter=ACMT --name-only \$GIT_PREVIOUS_COMMIT \$GIT_COMMIT)
                        docker login -u "\$HUB_USER" -p "\$HUB_PASS"
        
                        for i in \${CHANGED}; do
                            if [ \$i != "Jenkinsfile" ]; then
                                docker-app validate \${i}
                                docker-app push \${i}
                            fi
                        done
                    """
                }
            }
        }
    }

    post {
        always {
            withCredentials([string(credentialsId: 'teams-webhook-url', variable: 'WEBHOOK')]) {
                slackSend(
                        channel: 'nimbus',
                        message: "${env.JOB_NAME} - ${currentBuild.displayName} *${currentBuild.currentResult}* in ${currentBuild.durationString.replaceAll(' and counting', '')}" + ((currentBuild.currentResult != 'SUCCESS') ? " (<${env.BUILD_URL}console|Console>)" : ''),
                        color: (currentBuild.currentResult == 'SUCCESS') ? 'good' : 'danger'
                )
                office365ConnectorSend(
                        color:  (currentBuild.currentResult == 'SUCCESS') ? '00FF00' : 'FF0000',
                        message: "Build ${currentBuild.displayName} *${currentBuild.currentResult}* in ${currentBuild.durationString.replaceAll(' and counting', '')}",
                        webhookUrl: "${env.WEBHOOK}",
                        status: "${currentBuild.currentResult}"
                )
            }
        }

    }
}
