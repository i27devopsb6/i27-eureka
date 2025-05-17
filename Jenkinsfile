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
        DOCKER_HUB = "docker.io/i27devopsb6"
        DOCKER_CREDS = credentials('dockerhub_creds')
    }
    parameters {
        choice(name: 'scanOnly',
            choices: 'yes\nno',
            description: 'This will scan the application'
        )
        choice(name: 'buildOnly',
            choices: 'yes\nno',
            description: 'This will only build the application'
        )
        choice(name: 'dockerPush',
            choices: 'yes\nno',
            description: 'This will trigger docker build and docker push'
        )
        choice(name: 'deployToDev',
            choices: 'yes\nno',
            description: 'This will Deploy the application to Dev env '
        )
        choice(name: 'deployToTest',
            choices: 'yes\nno',
            description: 'This will Deploy the application to Test env '
        )
        choice(name: 'deployToStage',
            choices: 'yes\nno',
            description: 'This will Deploy the application to Stage env '
        )
        choice(name: 'deployToProd',
            choices: 'yes\nno',
            description: 'This will Deploy the application to Prod env '
        )
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
                echo "**************************************** Building Docker Image ****************************************"
                docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT ./.cicd
                echo "*** Listing Docker Images"
                docker images
                echo "**************************************** Docker Login ****************************************"
                docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
                echo "**************************************** Push Docker Image ****************************************"
                docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT
                """
            }
        }
        stage('Deploy to Dev') {
            steps {
                script {
                    // dockerDeploy(env, port)
                    dockerDeploy('dev','5761').call()
                }
            }
        }
        stage('Deploy to Test') {
            steps {
                script {
                    echo "Deploying to Test env"
                    dockerDeploy('tst','6761').call()
                }
            }
        }
        stage('Deploy to Stage') {
            steps {
                script {
                    echo "Deploying to stg env"
                    dockerDeploy('stg','7761').call()
                }
            }
        }
        stage('Deploy to prod') {
            steps {
                script {
                    echo "Deploying to prod env"
                    dockerDeploy('prd','8761').call()
                }
            }
        }
    }
}

def dockerDeploy(envDeploy, port) {
    return {
        echo "Deploying to $envDeploy env"
        withCredentials([usernamePassword(credentialsId: 'john_docker_cm_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
            script {
                try {
                    // stop the container
                    sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker stop ${APPLICATION_NAME}-$envDeploy\""
                    // remove the container
                    sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker rm ${APPLICATION_NAME}-$envDeploy\""
                }
                catch(err) {
                    echo "Error caught: $err"
                }
                sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no $USERNAME@$docker_vm_ip \"docker run --restart always --name ${APPLICATION_NAME}-$envDeploy -p $port:8761 -d ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:$GIT_COMMIT\""
            }
        }

    }
}
// eureka.i27cart.com
// dev env : hostport: 5761
// test env: hostport: 6761
// stage env: hostport: 7761
// prod: 8761

// for all the above container port is 8761