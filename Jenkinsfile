def mvn
def server = Artifactory.server 'artifactory'
def rtMaven = Artifactory.newMavenBuild()
def buildInfo
def DockerTag() {
	def tag = sh script: 'git rev-parse HEAD', returnStdout:true
	return tag
	}
pipeline {
  agent { label 'master' }
    tools {
      maven 'Maven'
      jdk 'JDK 1.11.*'
    }
  options { 
    timestamps () 
    buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '10', numToKeepStr: '5')	
// numToKeepStr - Max # of builds to keep
// daysToKeepStr - Days to keep builds
// artifactDaysToKeepStr - Days to keep artifacts
// artifactNumToKeepStr - Max # of builds to keep with artifacts	  
}	
  environment {
    SONAR_HOME = "${tool name: 'SonarQube Scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'}"
    DOCKER_TAG = DockerTag()	  
  }  
  stages {
    stage('Artifactory_Configuration') {
      steps {
        script {
		  rtMaven.tool = 'Maven'
		  rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server
		  buildInfo = Artifactory.newBuildInfo()
		  rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot', server: server
          buildInfo.env.capture = true
        }			                      
      }
    }
    stage('Execute_Maven') {
	  steps {
	    script {
		  rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
        }			                      
      }
    }	
    stage('War rename') {
	  steps {
		  	sh 'mv target/*.war target/helloworld.war'
        }			                      
      }
    stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'SonarQube Scanner' 
            }
            steps {
                withSonarQubeEnv(installationName: 'SonarServer') {
                    sh 'mvn sonar:sonar'
                }
            }
        }	
	stage('Quality_Gate') {
	  steps {
	    timeout(time: 3, unit: 'MINUTES') {
		  waitForQualityGate abortPipeline: true
        }
      }
    }
  stage('Build Docker Image'){
    steps{
      sh 'docker build -t 20152282/ansibledeploy:${DOCKER_TAG} .'
    }
  }	  	 
  stage('Docker Container'){
    steps{
      withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'docker_pass')]) {
	  sh 'docker login -u 20152282 -p ${docker_pass}'
      	  sh 'docker push 20152282/ansibledeploy:${DOCKER_TAG}'
	  }
    }
 }
    stage('Ansible Playbook'){
      steps {
       //ansiblePlaybook credentialsId: 'ans-server', inventory: 'inventory', playbook: 'ansibleplay.yml', tags: 'stop_container,delete_container'
       // Above command used to run the playbook with specified tags mentioned in the tags section.	
	 ansiblePlaybook credentialsId: 'ans-server', inventory: 'inventory', playbook: 'ansibleplay.yml'     
        }
      }	  	  
  }
}  
