// Starting the pipeline
pipeline {
  agent any
  // using maven from jenkins (name has to be the same as the one in the configuration)
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
        echo "Starting Your Pipeline Build"
      }
    }
   // Check for any unintentionally left credentials 
   stage ('GitCheckSecrets') {
      steps {
        sh 'touch trufflehog_results'
        sh 'docker run --rm dxa4481/trufflehog --regex --entropy=False --json https://github.com/devsecopss/webapp-pipeline.git > trufflehog_res.json || true'
        sh 'cat trufflehog_res.json' 
     
     //double Scan using git leaks
       sh 'docker run --rm /zricethezavgitleaks -r=https://github.com/devsecopss/webapp-pipeline  --pretty --verbose > res.json' //by default ouput json
       sh 'cat res.json'
     }  
    }
    // Third Parties Vulnerabilities Analysis
    stage ('Dependencies Analysis') {
      steps {
          sh '''
                dependency-check || true
             '''
          sh 'cat /var/lib/jenkins/workspace/webapp-pipeline/odc-reports/dependency-check-report.xml'

        // Double Check You never know  
        sh 'snyk test --json ---show-vulnerable-paths=all > snyk_res.json'
        sh 'snyk wizard'
        sh 'snyk monitor'
       }
    }
    
    stage ('Build') {
      steps {
      sh 'mvn clean package'
       }
    }
    
    // Static Code Analysis 
    stage ('SAST') {
        steps {
          withSonarQubeEnv('sonar') {
            sh 'mvn sonar:sonar'
            sh 'cat target/sonar/report-task.txt'
          }
        }
      }
    
     stage("Quality Gate"){
       steps {
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  slackSend channel: '#devsecopsdemo', message: 'Error with SAST'
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              } else {
                slackSend channel: '#devsecopsdemo', message: 'Pipeline Successfully passed SAST Verification'
              }
          }
        }
      }
    
    // Artifact Repository uploader to Nexus Server
    stage("Artifact Upload"){
      steps {
        echo "Add steps"
      }
    }
    


      stage ('Checking Services Health'){
        steps {
            sh 'echo Add my Script Over Here!!'
          }
      }

    stage ('Deploy-To-Tomcat') {
            steps {
               sshagent(['610d3050-5b62-4edc-8395-acddb916ec5c']) {
                    sh 'scp -o StrictHostKeyChecking=no target/*.war  yacine@webserver:/opt/tomcat/webapps/webapp-pipeline.war'
                   }
            }
   }
    
    stage ('DAST') {
      steps {
         sh ' docker run -t owasp/zap2docker-stable zap-baseline.py -t http://webserver/webapp-pipeline/ || true'
         //sh ' openvas cli '
      }
    }
    
    
    stage("Upload reports To Defect Dojo"){
      steps {
        echo "Uploading all steps reports to our vulnerability management tool Defect Dojo"
      }    
    }
    
  }
}
