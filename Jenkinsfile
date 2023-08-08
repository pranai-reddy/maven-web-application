def sendSlackNotifications(String buildStatus = 'STARTED') {
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


node{
    
    echo "Job Name is:  ${env.JOB_NAME}" // To print the Job Name
    echo "Node Name is : ${env.NODE_NAME}" // To print the Node Name
    
    //lly we can print all Env Variables based on requirement

    //Discard old builds and Poll SCM

    properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([pollSCM('* * * * *')])])
    
    try {

      sendSlackNotifications('STARTED')      

      //Get the code from Git and the Git Syntax can also be taken from Pipeline Syntax
    
    stage('CheckoutCode'){
        git 'https://github.com/pranai-reddy/maven-web-application.git'
    }
       
    //Maven Build, here bin/mvn is the maven location in Jenkins server. We have 2 Maven versions in Jenkins UI server, so we are specied which one to use(3.8.4)
    
    def mavenHome = tool name: 'maven 3.8.4'
    
    stage('Build'){
        sh "${mavenHome}/bin/mvn clean package"
    }
    
    // Sonar Report
    
    stage('ExecuteSonarQubeReport'){
        sh "${mavenHome}/bin/mvn sonar:sonar"
    }
    
    // Nexus Artifact
    
    stage('UploadArtifactintoNexus'){
        sh "${mavenHome}/bin/mvn deploy"
    }
    
    //Tomcat Deployment. Here Jenkins is running in different Server and Tomcat is running in Linux Redhat Server. We need to copy the war file from Jenkins server
    //to Redhat Linux Server where Tomcat is running. We need to specify the credentials of our Redhat Linux Server and which is a bad practic, so instead
    //we will use sshagent in Pipleine synatx and generate a Token and use the same.
    
    stage('DeployApplicationIntoTomcatServer'){
        
        
        sshagent(['8688a0c0-4127-48b4-b5ed-4e25c2946a8a']) {
            
            //Copy the war file from Jenkins Server to RedhatLinux server where Tomcat is reunning.
            // -o StrictHostKeyChecking=no is for if we are copying for the 1st time, it will ask for do you want to copy yes/no. This is used to avoid this.
            
            sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@3.111.42.83:/opt/apache-tomcat-9.0.75/webapps"
            
            

        }
    }

    }// Try block closure

  catch(e) {
 // If there was an exception thrown, the build failed
     currentBuild.result = "FAILED"
  }// Catch block closure
  finally {
    // Success or failure, always send notifications
    sendSlackNotifications(currentBuild.result)
  } //Finally block closure
}// Node block closure
