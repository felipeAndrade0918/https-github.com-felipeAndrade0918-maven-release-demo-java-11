#!groovy​

properties([
    [
        $class: 'BuildDiscarderProperty',
        strategy: [
            $class: 'LogRotator',
            artifactDaysToKeepStr: '1',
            artifactNumToKeepStr: '1',
            daysToKeepStr: '1',
            numToKeepStr: '5'
        ]
    ]
]);


node('docker-docker') {
	stage('Clean & Checkout') {
        deleteDir()
        checkout scm
	sh 'mvn clean'
	}
}
