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
        stage('Sonar') {
            steps {
                sh """
                echo "Starting Sonar Scan"
                mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=i27-eureka \
                    -Dsonar.host.url=http://34.45.105.80:9000 \
                    -Dsonar.login=sqp_60dac9ef2c3bae406aabee3d228c8495566e9c9c
                """
            }
        }
    }
}