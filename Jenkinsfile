pipeline {
    agent {
        label 'k8s-slave'
    }

    tools {
        maven 'maven-3.8.8'
        jdk 'jdk-17'
    }

    environment {
        APPLICATION_NAME = 'eureka'
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = "docker.io/kishoresamala84"
        DOCKER_CREDS = credentials('kishoresamala84_docker_creds')
        DOCKER_VM = '34.48.75.124'
    }

    stages {
        stage ('BUILD_STAGE') {
            steps {
                echo " ***** BUILD STAGE ***** "
                sh "mvn clean package -DskipTest=true"
                archiveArtifacts 'target/*.jar'
            }
        }

        stage ('SONARQUBE_STAGE') {
            steps {
                withSonarQubeEnv('sonarqube'){
                    script {
                        sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=i27-eureka-05 \
                        -Dsonar.host.url=http://34.86.250.120:9000 \
                        -Dsonar.login=sqa_7d01297a6e4c6d1d7f64e2f1137dcbc2df213ec4    
                        """                    
                    }
                }
                timeout (time: 2, unit: "MINUTES" ) {
                    waitForQualityGate abortPipeline: true
                }                
            }
        }

        stage ('BUILD_FORMAT_STAGE') {
            steps {
                script {
                    sh """
                    echo "Source JAR_FORMAT i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                    echo "Target JAR_FORMAT i27-${env.APPLICATION_NAME}-${BRANCH_NAME}-${currentBuild.number}.${env.POM_PACKAGING}"
                    """
                }
            }
        }

        stage ('DOCKER_BUILD_AND_PUSH') {
            steps {
                script {
                    sh "cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
                    sh "docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd"
                    sh "docker login -u ${env.DOCKER_CREDS_USR} -p ${env.DOCKER_CREDS_PSW}"
                    sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
                }   
            }
        }

        stage ('DEPLOT_TO_DEV') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'john_docker_vm_creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        echo " ********** *********** ********** ************ "
                    script {
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@'$DOCKER_VM' \"docker images\""
                        echo " ********** *********** ********** ************ "
                    }
                }
            }
        }
    }
}