pipeline {
  agent any
  tools {
    maven 'Maven'
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
    
   stage ('GitCheckSecrets') {
      steps {
        sh 'rm trufflehog_results || true'
        sh 'docker run dxa4481/trufflehog --json https://github.com/DevOpsInterner/webapp-pipeline.git > trufflehog_results'
        sh 'cat trufflehog_results' 
       }
    }
    
    stage ('Dependencies Analysis') {
      steps {
          sh 'rm owasp* || true'
          sh '''
                  FILE=dependency-check.sh
                  if ! test -f "$FILE"; then
                     wget "https://raw.githubusercontent.com/DevOpsInterner/webapp-pipeline/master/dependency-check.sh"
                  fi
                  chmod +x dependency-check.sh
                  bash dependency-check.sh
             '''
          sh 'cat /var/lib/jenkins/workspace/webapp-pipeline/odc-reports/dependency-check-report.xml'
       }
    }
    
    stage ('SAST') {
        steps {
          withSonarQubeEnv('sonar') {
            sh 'mvn sonar:sonar'
            sh 'cat target/sonar/report-task.txt'
          }
        }
      }

    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
    
    stage ('Deploy-To-Tomcat') {
            steps {
              sh 'scp -o StrictHostKeyChecking=no target/*.war  /opt/apache-tomcat-9.0.33/webapps/webapp-pipeline.war'
              }         
    }
    
    stage ('DAST') {
      steps {
         sh ' docker run -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.1.10:4455/webapp-pipeline/ || true'
      }
    }
    
    
  }
}
