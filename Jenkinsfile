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
                withSonarQubeEnv(sonarqube){
                    script {
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=i27-eureka-05 \
                        -Dsonar.host.url=http://34.86.250.120:9000 \
                        -Dsonar.login=sqa_7d01297a6e4c6d1d7f64e2f1137dcbc2df213ec4                        
                    }
                    timeout (time: 2, unit: "MINUTES" ) {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

        stage ('BUILD_FORMAT_STAGE') {
            steps {
                script {
                    sh "Source JAR_FORMAT i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                    sh "Target JAR_FORMAT i27-${env.APPLICTION_NAME}-${BRANCH_NAME}-${currentBuild.number}-${GIT_COMMIT}"
                }
            }
        }
    }
}