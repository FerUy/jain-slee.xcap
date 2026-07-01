pipeline {
	agent any
	tools {
        jdk 'jdk-11'
        maven 'maven-3.9.12'
    }
	options {
    	buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))
  	}
	parameters {
		string(name: 'JSLEE_XCAP_VERSION', defaultValue: '7.2.0', description: 'The major version for JAIN SLEE XCAP')
	}
	environment {
        XCAP_BUILD_VERSION = "${params.JSLEE_XCAP_VERSION}-${BUILD_NUMBER}"
    }
	stages {
		stage("Build") {
			steps {
				script {
                    XCAP_BUILD_VERSION = "${params.JSLEE_XCAP_VERSION}-${BUILD_NUMBER}"
                    currentBuild.displayName = "#${XCAP_BUILD_VERSION}"
                    currentBuild.description = "JAIN SLEE XCAP (${env.BRANCH_NAME})"
                }
                sh "mvn clean install -Dmaven.test.skip=true"
			}
		}
		stage('Set Version') {
			steps {
				sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${XCAP_BUILD_VERSION}"
                sh "mvn versions:commit"
			}
		}
		stage("Release") {
			steps {
                sh "mvn clean install -Prelease -Drelease.dir=../../../${XCAP_BUILD_VERSION} -Dmaven.test.skip=true"
			}
		}
		stage('Zip Resources') {
			steps {
				dir("${XCAP_BUILD_VERSION}/resources") {
					sh "zip -r xcap.zip xcap-client"
					sh 'rm -rf xcap-client'
				}
			}
		}
		stage('Save Artifacts') {
            steps {
                archiveArtifacts artifacts: "${XCAP_BUILD_VERSION}/", followSymlinks: false, onlyIfSuccessful: true
            }
		}
        stage('Push to Repo') {
            when{anyOf {branch 'master'; branch 'release'}}
			steps {
                sh "mkdir -p /var/www/html/NAIKERI/jain-slee.xcap/${XCAP_BUILD_VERSION}/"
                sh "cp -r ${XCAP_BUILD_VERSION}/ /var/www/html/NAIKERI/jain-slee.xcap/${XCAP_BUILD_VERSION}/"
				sh "rm -rf ${XCAP_BUILD_VERSION}"
			}
		}
    }
	post {
		success { echo "JAIN-SLEE XCAP successfully built" }
		failure { echo "Building JAIN-SLEE XCAP failed" }
	}
}
