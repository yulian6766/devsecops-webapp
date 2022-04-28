pipeline {
  agent any
  
  tools {
    maven 'Maven'
  }
  
  stages {
    stage ('Initialize'){
	  steps {
	    sh '''
		      echo "PATH = $(PATH)"
			  echo "M2_HOME = $(M2_HOME)"
		   '''
	  }
	}
	
	stage ('Build'){
	  steps {
	    sh 'mvn clean package'
	  }
	}
	
	stage ('Deploy-To-Staging') {
          steps {
            sshagent(['stagingServer']) {
              sh 'scp -o StrictHostKeyChecking=no target/*.war vagrant@192.168.99.26:/prod/apache-tomcat/webapps/webapp.war'
            }      
          }       
        }
  }
}
