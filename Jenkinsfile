pipeline {
    agent any
    tools{
        jdk  'jdk17'
        maven  'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '15fb69c3-3460-4d51-bd07-2b0545fa5151', poll: false, url: 'https://github.com/jaiswaladi246/Shopping-Cart.git'
            }
        }
        
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }

         stage('Trivy FS Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Shopping-Cart '''
               }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }

          # Maven pipeline Integration (install this tool to create pipeline syntax)
         stage('Deploy To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings-xml') { 
                sh "mvn deploy -DskipTests=true"
                }
            }
        }

       
       stage('Build & Tag Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t kubegourav/shopping-cart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy image kubegourav/shopping-cart:latest > trivy-report.txt"
            }
        }

       stage('Push Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push kubegourav/shopping-cart:latest"
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker run -d --name= Ekart -p 8070:8070 kubegourav/shopping-cart:latest"
                    }
                }
            }
        }

        
        }   
    }

