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
    
   stage ('GitCheckSecrets') {
      steps {
        sh 'touch trufflehog_results'
        sh 'docker run --rm --name th dxa4481/trufflehog --regex --entropy=False --json https://github.com/devsecopss/webapp-pipeline.git > trufflehog_results'
        sh 'cat trufflehog_results' 
       }
    }
    
    stage ('Dependencies Analysis') {
      steps {
          sh '''
                dependency-check || true
                snyk test
                snyk monitor
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
              sh 'scp -o StrictHostKeyChecking=no target/*.war  yacine@webserver:/opt/tomcat/webapps/webapp-pipeline.war'
              }         
    }
    
    stage ('DAST') {
      steps {
         sh ' docker run -t owasp/zap2docker-stable zap-baseline.py -t http://webserver/webapp-pipeline/ || true'
      }
    }
    
    
  }
}
