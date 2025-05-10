// This pipeline is for Eureke deployment 
pipeline {
    agent {
        label 'k8s-slave'
    }
    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }
    environment {
        APPLICATION_NAME = "eureka"
    }
    stages {
        stage('Build') {
            steps {
                echo "Building ${env.APPLICATION_NAME} Application"
                sh "mvn clean package -DskipTests=true" 
                archive 'target/*.jar'
            }
        }
        stage('Unit tests') {
            steps {
                echo "Performing Unit tests for ${env.APPLICATION_NAME} Application"
                sh 'mvn test'
            }
        }
    }
}