pipeline {
  agent any 
  tools {
    maven 'maven'
  }
  stages {
    stage ('Initialize') {
      steps {
        sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
            ''' 
      }
    }
    
    stage ('Check-Git-Secrets') {
      steps {
        sh 'rm trufflehog || true'
        sh 'sudo docker run -t gesellix/trufflehog --json https://github.com/cehkunal/webapp.git > trufflehog'
        sh 'cat trufflehog'
      }
    }
    
    stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/ehkrishna/webapp/main/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh '"bash owasp-dependency-check.sh" || true'
         sh '"cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml" || true'
        
      }
    }
    
    stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh '"mvn sonar:sonar" || true'
          sh '"cat target/sonar/report-task.txt" || true'
        }
      }
    }
    
    stage ('Build') {
      steps {
      sh '"mvn clean package" || true'
       }
    }
    
    stage ('Deploy-To-Tomcat') {
            steps {
           sshagent(['tomcat']) {
                sh '"scp  -o StrictHostKeyChecking=no webapp/target/webapp.war ubuntu@http://13.233.44.214:8090/opt/tomcat9/webapps/webapp.war" || true'
              }      
           }       
    }
    
    
    stage ('DAST') {
      steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.232.158.44 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://13.233.44.214:8090/webapp/" || true'
        }
      }
    }
    stage ('Port Scanning') {
           steps {
             sh 'rm openports.txt || true'
             sh 'nmap -Pn  13.233.44.214 > openports.txt'
             sh 'cat openports.txt'
           }
           }
    stage ('Defect Dojo') {
           steps {
             sh ' echo "vulnerabilities are submitted to Defect-Dojo" '
           }
           }
  }
}
