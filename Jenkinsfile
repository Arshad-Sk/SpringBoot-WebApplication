pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout ') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Arshad-SK/SpringBoot-WebApplication.git'
            }
        }
        
        stage('Code Compile') {
            steps {
                    sh "mvn compile"
            }
        }
        
        stage('Run Test Cases') {
            steps {
                    sh "mvn test"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Java-WebApp \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Java-WebApp '''
    
                }
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                   dependencyCheck additionalArguments: '--scan ./   ', odcInstallation: 'DP-Check'
                   dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Maven Build') {
            steps {
                    sh "mvn clean package"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: 'b9f5bd82-bfa9-4e28-a440-6792e37b103e', toolName: 'docker') {
                            sh "docker build -t webapp ."
                            sh "docker tag webapp sk77arshad/webapp:latest"
                            sh "docker push sk77arshad/webapp:latest "
                        }
                   } 
            }
        }
        
        stage('Docker Image scan') {
            steps {
                    sh "trivy image sk77arshad/webapp:latest "
            }
        }
        
          stage('Docker Container') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: 'b9f5bd82-bfa9-4e28-a440-6792e37b103e', toolName: 'docker') {

                            sh "docker run -d --name webapp -p 8080:8080 sk77arshad/webapp:latest"
                        }
                   } 
            }
        }
        
    }
}
