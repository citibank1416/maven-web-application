pipeline{
    agent any

    tools {
       maven 'maven3.8.5'
       git 'Default'
     }
     options {
      timestamps()
      buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')
    }

    triggers {
      pollSCM ('* * * * *')
      //cron ('* * * *  *')
    }
    

   
         stages{
             stage('codecheckout'){
                steps{
                    notifyBuild("STARTED")
                    git branch: 'development', credentialsId: '0e022228-9245-4ce5-a433-2533784b1430',
                    url: 'https://github.com/citibank1416/maven-web-application.git'
                 }
            }
        stage('build'){
            steps{
                sh "mvn clean package"
            }
        }
        stage('sonarqubereport'){
            steps{
                sh "mvn sonar:sonar"
            }
        }
        stage('uploadtonexus'){
            steps{
                sh "mvn clean deploy"
            }
        }
        
        stage('deploytotomcat'){
            steps{
                 sshagent(['9fa038e4-5806-4f0f-a65a-3f691b5dba9f']) {
                     sh "scp -o StrictHostKeyChecking = no /target/maven-web-application.war
                         ec2-user@13.234.202.224:/opt/apache-tomcat/webapps"
                }
            }
        } 
          
     } //stages end
    
    post {
         aborted {
           notifyBuild(currentBuild.result)
        }
        success {
          notifyBuild(currentBuild.result)
       }
        failure {
            notifyBuild(currentBuild.result)
        }
        fixed {
          notifyBuild(currentBuild.result)
        }
    }

} //pipeline end

//slack notification
def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
