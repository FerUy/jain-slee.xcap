pipeline {
    agent any

	tools {
	    jdk 'JDK 11'
		maven 'Maven_3.8.5'
	}
	parameters { string(name: 'JAIN_SLEE_XCAP_MAJOR_VERSION', defaultValue: '7.2.0', description: 'The major version for JAIN SLEE XCAP version') }

    environment{
        XCAP_BUILD_VERSION = "${params.JAIN_SLEE_XCAP_MAJOR_VERSION}-${BUILD_NUMBER}"
    }

	stages{
		stage("Build ") {
			steps {
			    sh 'java -version'
				script {
				    XCAP_BUILD_VERSION = "${params.JAIN_SLEE_XCAP_MAJOR_VERSION}-${BUILD_NUMBER}"
                    currentBuild.displayName = "#${XCAP_BUILD_VERSION}"
                    currentBuild.description = "JAIN SLEE XCAP (${env.BRANCH_NAME})"
                }
				echo "Building JAIN SLEE XCAP version (#${XCAP_BUILD_VERSION})"
                sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${XCAP_BUILD_VERSION} clean install -DskipTests"
				sh 'mvn clean install -DskipTests'
                echo "Maven build completed."
            }
        }

		stage("Release") {
			steps {
				echo "Building a wildfly release version of #${XCAP_BUILD_VERSION}"
                sh 'find . -type d -name \\"target\\" -exec r -r {} +'
                sh "mvn versions:set -DnewVersion=${XCAP_BUILD_VERSION} clean install -Prelease -Drelease.dir=../../../${XCAP_BUILD_VERSION} -Dmaven.test.skip=true"
                echo "Building wildfly version completed."
            }
        }
	}
}