pipeline {
    agent {
        label 'docker'
    }

    options {
        skipDefaultCheckuot()
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                parallel(one: {
                    dir('sync-endpoint') {
                        checkout scm
                    }
                }, two: {
                    dir('sync-endpoint-container') {
                        script {
                            checkoutGitRepo('https://github.com/linl33/sync-endpoint-containers.git')
                        }
                    }
                })
            }
        }
    }
}

Map checkoutGitRepo(String url) {
    checkout scm: [$class: 'GitSCM', branches: [[name: '*/master']],
                   doGenerateSubmoduleConfigurations: false,
                   extensions: [[$class: 'WipeWorkspace'], [$class: 'CloneOption', honorRefspec: true, noTags: true, reference: '', shallow: true]],
                   submoduleCfg: [], userRemoteConfigs: [[url: url]]]
}