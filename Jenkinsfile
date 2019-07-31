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


node('k8s-maven') {
	stage('Clean & Checkout') {
        deleteDir()
        checkout scm
	sh 'mvn clean install'
	}
}
