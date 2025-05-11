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
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        APPLICATION_NAME = "eureka"
        SONAR_URL = "http://34.45.105.80:9000"
        SONAR_TOKEN  = credentials('sonar_creds')
    }
    stages {
        stage('Build') {
            steps {
                echo "Building ${env.APPLICATION_NAME} Application"
                sh "mvn clean package -DskipTests=true" 
                archive 'target/*.jar'
            }
        }
        // stage('Unit tests') {
        //     steps {
        //         echo "Performing Unit tests for ${env.APPLICATION_NAME} Application"
        //         sh 'mvn test'
        //     }
        // }
        stage('Sonar') {
            steps {
                withSonarQubeEnv('SonarQube'){
                    sh """
                        echo "Starting Sonar Scan"
                        mvn clean verify sonar:sonar \
                            -Dsonar.projectKey=i27-eureka \
                            -Dsonar.host.url=${env.SONAR_URL} \
                            -Dsonar.login=${env.SONAR_TOKEN}
                    """
                }
                timeout(time: 2, unit: "MINUTES") {
                    script {
                        waitForQualityGate  abortPipeline: true
                    }
                }
            }
        }
        stage ('Build Format') {
            steps {
                // Existing: i27-eureka-0.0.1-SNAPSHOT.jar
                // Desination: i27-eureka-BUILD_NUMBER-branchName.jar
                script {
                    echo "Testing JAR Source: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                    echo "Testing Jar Destination: i27-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"
                }
            }
        }
        stage ('Docker Build') {
            steps {
                sh """
                ls -la
                cp ${workspace}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
                ls -la ./.cicd
                """
            }
        }
    }
}