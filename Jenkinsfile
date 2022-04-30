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
	
	stage ('Check-Git-Secrets'){
	  steps {
	    sshagent(['toolServer']){
		  sh 'ssh -t -t vagrant@192.168.99.28 -o StrictHostKeyChecking=no "rm trufflehog || true; docker pull gesellix/trufflehog; docker run gesellix/trufflehog --json  https://github.com/yulian6766/devsecops-webapp.git > trufflehog; cat trufflehog"'
		}
	  }
	}
	
	stage ('Dependency-Vuln-Check'){
	  steps {
	    sshagent(['toolServer']) {
	      sh '''ssh -o StrictHostKeyChecking=no vagrant@192.168.99.28 "rm owasp* || true;
	          wget "https://raw.githubusercontent.com/yulian6766/devsecops-webapp/master/owasp-dependency-check.sh";
		      chmod +x owasp-dependency-check.sh;
		      sh owasp-dependency-check.sh;
		      cat /home/vagrant/OWASP-Dependency-Check/reports/dependency-check-report.xml";
		     '''
		}
	  }
	}
	
	stage ('SAST') {
	  steps {
	    withSonarQubeEnv ('Sonar') {
		  sh 'mvn sonar:sonar'
		  sh 'cat target/sonar/report-task.txt'
		}
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
	
	stage ('DAST'){
	  steps {
	    sshagent(['toolServer']) {
		  sh 'ssh -o StrictHostKeyChecking=no vagrant@192.168.99.28 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.99.26:8080/webapp/"'
		}
	  }
	}
	
  }
}
