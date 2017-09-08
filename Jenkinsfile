final String syncEndpointDir = 'sync-endpoint'
final String syncEndpointContainerDir = 'sync-endpoint-container'

pipeline {
    agent {
        label 'docker'
    }

    options {
        skipDefaultCheckout()
        skipStagesAfterUnstable()
        buildDiscarder(logRotator(numToKeepStr:'5'))
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

        stage('Install Dependencies') {
            steps {
                dir(syncEndpointDir + '/src/main/libs') {
                    sh 'ant'
                }
            }
        }

        stage('Build WAR') {
            steps {
                dir(syncEndpointDir) {
                    // delete gwt files to speed up compilation
                    // TODO: remove after https://github.com/opendatakit/sync-endpoint/pull/4
                    sh 'find . -name "*.gwt.xml" -type f -delete'

                    sh 'sed -i "s/odk-mysql-it-settings/odk-container-settings/" aggregate-mysql/pom.xml'
                    sh 'mvn -pl "aggregate-src, odk-container-settings, aggregate-mysql" package'

                    sh "mv aggregate-mysql/target ../${syncEndpointContainerDir}/target"
                }
            }
        }

        stage('Build Image') {
            steps {
                dir(syncEndpointContainerDir) {
                    script {
                        docker.build("odk/sync_endpoint:${env.BUILD_NUMBER}", '--no-cache --pull -f Dockerfile.dev .')
                    }
                }
            }
        }

        stage('Image Check') {
            steps {
                sh 'docker images'
            }
        }
    }

    post {
        success {
            script {
                docker.image("odk/sync_endpoint:${env.BUILD_NUMBER}").tag('latest')
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