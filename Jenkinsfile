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
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/dheeraj7444/Ekart.git'
            }
        }
        stage('Compile') {
            steps {
                sh'mvn compile'
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh'trivy fs .'
            }
        }
        stage('Build') {
            steps {
                sh'mvn package -DskipTests=true'
            }
        }
        stage('OWASP FS SCAN') {
            steps {
               dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
               dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        } 
        stage('SONARQUBE ANALYSIS') {
            steps {
             withSonarQubeEnv('sonar') {
             sh " $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Ekart -Dsonar.java.binaries=. -Dsonar.projectKey=Ekart "
   
               }  
            }
        }
        stage('Deploy on Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'globle-setting-xml') {
                sh'mvn deploy -DskipTests=true'
                }
            }
        }
        stage('Build and tag Docker Image') {
            steps {
               withDockerRegistry(url: 'https://index.docker.io/v1/', credentialsId: 'docker-cred') {
                  sh "docker build -t shopping-cart:${BUILD_ID} -f docker/Dockerfile ."
                  sh"docker tag shopping-cart:${BUILD_ID} dheeraj7444/shopping-cart:${BUILD_ID}"
                } 
            }
        }
        stage('Docker Push') {
            steps {
               withDockerRegistry(url: 'https://index.docker.io/v1/', credentialsId: 'docker-cred') {
                   sh"docker push dheeraj7444/shopping-cart:${BUILD_ID}"
                } 
            }
        }
    }
        
}
