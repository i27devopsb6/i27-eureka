// This pipeline is for Eureke deployment 
pipeline {
    agent {
        label 'k8s-slave'
    }
    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }
    stages {
        stage('Build') {
            steps {
                echo "Building Eureka Application"
                sh "mvn clean package -DskipTests=true" 
            }
        }
    }
}