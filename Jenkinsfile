final String syncEndpointDir = 'sync-endpoint'
final String syncEndpointContainerDir = 'sync-endpoint-container'

pipeline {
    agent {
        label 'docker'
    }

    options {
        skipDefaultCheckout()
    }

    tools {
        jdk 'JDK 1.8'
    }

    stages {
        stage('Env Check') {
            steps {
                echo JAVA_HOME
                sh 'env'
            }
        }

//        stage('Unset Env') {
//            steps {
//                sh 'unset JAVA_HOME'
//            }
//        }

        stage('Checkout') {
            steps {
                parallel(one: {
                    dir(syncEndpointDir) {
                        checkout scm
                    }
                }, two: {
                    dir(syncEndpointContainerDir) {
                        script {
                            checkoutGitRepo('https://github.com/linl33/sync-endpoint-containers.git', 'master')
                        }
                    }
                })
            }
        }

        stage('Install Ant Dependency') {
            steps {
                dir(syncEndpointDir + '/src/main/libs') {
                    sh 'ant'
                }
            }
        }

        stage('Build Sync Endpoint') {
            steps {
                dir(syncEndpointDir) {
                    sh 'mvn -pl "aggregate-src" package'
                }
            }
        }
    }
}

Map checkoutGitRepo(String url, String branch) {
    checkout scm: [$class: 'GitSCM', branches: [[name: '*/' + branch]],
                   doGenerateSubmoduleConfigurations: false,
                   extensions: [[$class: 'WipeWorkspace'], [$class: 'CloneOption', honorRefspec: true, noTags: true, reference: '', shallow: true]],
                   submoduleCfg: [], userRemoteConfigs: [[url: url]]]
}