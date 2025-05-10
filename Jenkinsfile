// This pipeline is for Eureke deployment 
pipeline {
    agent {
        label 'k8s-slave'
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