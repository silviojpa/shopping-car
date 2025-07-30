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
                    withDockerRegistry(credentialsId: 'f45e0e3c-4b75-4952-9ab0-c00ff2c9b820', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart silvio69luiz/shopping-cart:latest"
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'f45e0e3c-4b75-4952-9ab0-c00ff2c9b820', toolName: 'docker') {
                        sh "docker push silvio69luiz/shopping-cart:latest"
                    }
                }
            }
        }
        
        stage('Deploy To Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'f45e0e3c-4b75-4952-9ab0-c00ff2c9b820', toolName: 'docker') {
                        sh "docker run -d --name shopping -p 8070:8070 silvio69luiz/shopping-cart:latest"
                    }
                }
            }
        }
    }
}
