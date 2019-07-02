
def project
def branch
def version

pipeline {
    agent any
    environment {
        project = ""
        branch  = ""
        version = ""
    }

    stages {
        stage('Checkout') {
            steps {
                script{
                    project = artifactId()
                    branch  = env.BRANCH_NAME
                    version = versionByBranch( branch  )
                }
                echo "Checkout branch -> ${branch}"
           		checkout scm
       	    	checkout([$class: 'GitSCM', branches: [[name: '*/' + branch]], extensions: [[$class: 'CleanCheckout'],[$class: 'LocalBranch', localBranch: branch]]])
   	    	    changeVersionPom( version, branch )
            }
        }
        stage('Build') {
            steps {
                echo 'Clean Build'
                sh 'mvn clean compile'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing'
                sh 'mvn test'
            }
        }
        stage('JaCoCo') {
            steps {
                echo 'Code Coverage'
                jacoco()
            }
        }
        stage("Sonar analysis") {
            steps {
              withSonarQubeEnv('pct-sonar') {
                 echo "Sonar version: ${version}"
                sh "mvn sonar:sonar -DskipTests -Dsonar.projectVersion=${version}"
              }
            }
        }

        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }

        stage('Release deploy') {
            when { expression { return branch == 'master'} }
            steps {
                echo "Release deploy branch : ${branch}"
                releaseDeploy()
            }
        }

        stage('Snapshot deploy ') {
            when { not { expression { return branch == 'master'} } }
            steps {
                echo "Snapshot deploy branch : ${branch}"
                snapshotDeploy()
            }
        }
    }

    post {
        always {
            echo 'JENKINS PIPELINE'
            archive "target/**/*"
            junit 'target/surefire-reports/*.xml'
        }
        success {
            echo 'JENKINS PIPELINE SUCCESSFUL'
        }
        failure {
            echo 'JENKINS PIPELINE FAILED'
        }
        unstable {
            echo 'JENKINS PIPELINE WAS MARKED AS UNSTABLE'
        }
        changed {
            echo 'JENKINS PIPELINE STATUS HAS CHANGED SINCE LAST EXECUTION'
        }
    }
}

def releaseDeploy(){

    //def commitid = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    //def workspacePath = pwd()
    //sh "echo ${commitid} > ${workspacePath}/expectedCommitid.txt"
    //-Dcommitid=${commitid}
    sh "mvn -Dresume=false -DskipTests release:prepare release:perform -Darguments='-Dmaven.javadoc.skip=true'"
}

def snapshotDeploy(){

    def commitid = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    def workspacePath = pwd()
    sh "echo ${commitid} > ${workspacePath}/expectedCommitid.txt"
    sh "mvn deploy -DskipTests"
}

def versionByBranch( branchName ){
    def (vNumber) = version().tokenize('-')

    if (branchName == 'master') {
        return vNumber
	} else{
        return "${vNumber}-${branchName}-SNAPSHOT"
    }
}

def changeVersionPom( version, branch ){

    if ( branch != 'master' ) {
        //renomear a versao do pom
        sh "mvn -B org.codehaus.mojo:versions-maven-plugin:2.5:set -DprocessAllModules -DnewVersion=${version} "
    }
}

def version() {
    def matcher = readFile('pom.xml') =~ '<version>(.+)-.*</version>'
    matcher ? matcher[0][1] : null
}

def artifactId() {
    def matcher = readFile('pom.xml') =~ '<artifactId>(.+)</artifactId>'
    matcher ? matcher[0][1] : null
}