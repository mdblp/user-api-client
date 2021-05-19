#!groovy
@Library('mdblp-library') _
pipeline {
    agent any
    triggers {
        //Check for modifications in Bitbucket every hours
        //This is the condition for the bitbucket webhook trigger to work
        pollSCM('30 * * * *')
    }
    environment {
        JUNIT_REPORT_PATH = "tests/report.xml"
    }
    stages {
        stage('Build & package') {
            agent {
                dockerfile {
                    filename 'jenkins-build.dockerfile'
                    reuseNode true
                }
            }
            steps {
                withCredentials([string(credentialsId: 'nexus-token', variable: 'NEXUS_TOKEN')]) {
                    sh 'npm install'
                    sh 'npm run build'
                    stash name: "test", includes: "**"
                    //we could execute npm version to automatically update the version
                    // make security check and lint
                    sh 'npm run security-check && npm run lint'
                }
                stash name: utils.packStashName, includes: "dist/**"

            }
        }
        stage('Running tests') {
            agent {
                docker {
                    image 'node:12-alpine'
                }
            }
            steps {
                dir("test-dir") {
                    unstash "test"
                }
                sh '''
                    cd test-dir
                    npm run test
                '''
            }
        }
        stage('Publish') {
            when { branch "master" }
            steps {
                publish()
            }
        }
   }

}