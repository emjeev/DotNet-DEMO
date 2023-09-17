pipeline {
    agent any
    
    tools {
        jdk 'JDK'
    }
    
      environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/emjeev/DotNet-DEMO.git'
            }
        }
        
        stage('OWASP Dependency') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy FS') {
            steps {
               sh "trivy fs ."
            }
        }
        
        stage(" Sonarqube Analysis "){
            steps{
                 withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Dotnet-demo \
                    -Dsonar.projectKey=Dotnet-demo '''
                    
                 }
            }
        } 
        
        stage('Docker build and tag') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "make image"
                    }
               }
            }
        }
        stage("Trivy Image Scan "){
            steps{
                sh "trivy image emjeev82/dotnet-demoapp "
            }
        } 
         
        stage('Docker Push') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "make push"
                    }
               }
            }
        }
        
        stage('Deloy to container ') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker run -d -p 5000:5000  emjeev82/dotnet-demoapp "
                    }
               }
            }
        }
    }
