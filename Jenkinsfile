pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/silviojpa/shopping-car.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.url=http://localhost:9000/ \
                    -Dsonar.login=squ_aa2b36e6f567f703ad1cd00fd7bb64c6a55dc160 \
                    -Dsonar.projectName=shopping-car \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=shopping-car
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan target/', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'c9b058e5-bfe6-41f8-9b5d-dc0b0d2955ac', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart adijaiswal/shopping-cart:latest"
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'c9b058e5-bfe6-41f8-9b5d-dc0b0d2955ac', toolName: 'docker') {
                        sh "docker push adijaiswal/shopping-cart:latest"
                    }
                }
            }
        }
        
        stage('Deploy To Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'c9b058e5-bfe6-41f8-9b5d-dc0b0d2955ac', toolName: 'docker') {
                        sh "docker run -d --name shopping -p 8070:8070 adijaiswal/shopping-cart:latest"
                    }
                }
            }
        }
    }
}
