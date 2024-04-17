pipeline {
    agent any
    
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages {
        stage("Git Checkout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/lokesh2123/Petclinic.git'
            }
        }
        
        stage("Compile"){
         steps{
            sh "mvn clean compile"
            }
        }
        
        stage("Test Cases"){
            steps{
             sh "mvn test -DskipTests=true"
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
                }
            }
        }
        
        stage('Maven Package') {
            steps {
            sh 'mvn clean package'
            }
        }
        
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan target/', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Push to Nexus Repo') {
            steps {
            configFileProvider([configFile(fileId: '1516a7c5-712d-43f5-84d4-a820cca3eb56', variable: 'mavensettings')]) {
                sh "mvn -s $mavensettings clean deploy -DskipTests=true"
                }
            }
        }
        stage("Deploy To Tomcat"){
            steps{
                sh "cp  /root/.jenkins/workspace/My-pipeline/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps "
            }
        }
    }
}
