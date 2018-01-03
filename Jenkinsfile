//#!groovy
import com.bit13.jenkins.*

properties ([
	buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5')),
	disableConcurrentBuilds(),
	pipelineTriggers([
		pollSCM('H/30 * * * *')		
	]),
])


if(env.BRANCH_NAME ==~ /master$/) {
	return
}


node ("arduino") {
	def ProjectName = "alunar-prusa-i3-marlin-i3-firmware"
	def BOARD_ID = "arduino:avr:mega:cpu=atmega2560"
	def INO_PATH = "${WORKSPACE}/Marlin_I3/Marlin_I3.ino"

	def slack_notify_channel = null

	def SONARQUBE_INSTANCE = "bit13"

	def MAJOR_VERSION = 1
	def MINOR_VERSION = 0

	env.PROJECT_MAJOR_VERSION = MAJOR_VERSION
	env.PROJECT_MINOR_VERSION = MINOR_VERSION
	
	env.CI_BUILD_VERSION = Branch.getSemanticVersion(this)
	env.CI_DOCKER_ORGANIZATION = Accounts.GIT_ORGANIZATION
	currentBuild.result = "SUCCESS"
	def errorMessage = null


	wrap([$class: 'TimestamperBuildWrapper']) {
		wrap([$class: 'AnsiColorBuildWrapper', colorMapName: 'xterm']) {
			Notify.slack(this, "STARTED", null, slack_notify_channel)
			try {
					stage ("install" ) {
							deleteDir()
							Branch.checkout(this, ProjectName)
							Pipeline.install(this)
					}
					stage ("build") {
						echo "in build..."
						sh script: """#!/usr/bin/env bash
						set -e;
						mkdir -p ${WORKSPACE}/dist;
						# temp full path
						/usr/local/bin/arduino/arduino-builder \
							--compile \
							-hardware /usr/local/bin/arduino/hardware \
							-tools /usr/local/bin/arduino/hardware/tools \
							-tools /usr/local/bin/arduino/tools-builder \
							-fqbn ${BOARD_ID} \
							-build-path '${WORKSPACE}/dist' \
							'${INO_PATH}';
						ls -lFa ${WORKSPACE}/dist;"""
					}
					stage ("test") {
					}
					stage ("deploy") {
							// sh script: "${WORKSPACE}/.deploy/deploy.sh -n '${ProjectName}' -v '${env.CI_BUILD_VERSION}'"
					}
					stage ('cleanup') {
							// this only will publish if the incominh branch IS develop
							Branch.publish_to_master(this)
							Pipeline.cleanup(this)
					}
			} catch(err) {
				currentBuild.result = "FAILURE"
				errorMessage = err.message
				throw err
			}
			finally {
				if(currentBuild.result == "SUCCESS") {
					if (Branch.isMasterOrDevelopBranch(this)) {
						currentBuild.displayName = "${env.CI_BUILD_VERSION}"
					} else {
						currentBuild.displayName = "${env.CI_BUILD_VERSION} [#${env.BUILD_NUMBER}]"
					}
				} else {
					Notify.gntp(this, "${ProjectName}: ${currentBuild.result}", errorMessage)
				}
				Notify.slack(this, currentBuild.result, errorMessage)
			}
		}
	}
}
