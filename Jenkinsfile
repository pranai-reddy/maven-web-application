node{
    
    echo "Job Name is:  ${env.JOB_NAME}" // To print the Job Name
    echo "Node Name is : ${env.NODE_NAME}" // To print the Node Name
    
    //lly we can print all Env Variables based on requirement
    
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
            
            sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@3.110.207.157:/opt/apache-tomcat-9.0.75/webapps"
            
            

        }
    }
}
