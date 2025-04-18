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
        POM_VERSION = ''
        POM_PACKAGING = ''
        DOCKER_HUB = "docker.io/kishoresamala84"
        DOCKER_CREDS = credentials('kishoresamala84_docker_creds')
        DOCKER_VM = '34.48.75.124'
    }

    stages {
        stage('INIT_POM_VALUES') {
            steps {
                script {
                    def pom = readMavenPom()
                    env.POM_VERSION = pom.version
                    env.POM_PACKAGING = pom.packaging
                }
            }
        }

        stage('BUILD_STAGE') {
            steps {
                echo "***** BUILD STAGE *****"
                sh "mvn clean package -DskipTests=true"
                archiveArtifacts 'target/*.jar'
            }
            post {
                success {
                    script {
                        emailext(
                            subject: "Success: Build Stage for ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                            body: "Build Stage Passed.\n\nBuild Number: ${env.BUILD_NUMBER}\n\nJob URL: ${env.BUILD_URL}",
                            to: 'kishorecloud.1725@gmail.com'
                        )
                    }
                }
                failure {
                    script {
                        emailext(
                            subject: "Failure: Build Stage for ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                            body: "Build Stage Failed.\n\nBuild Number: ${env.BUILD_NUMBER}\n\nJob URL: ${env.BUILD_URL}",
                            to: 'kishorecloud.1725@gmail.com'
                        )
                    }
                }
            }
        }

        stage('SONARQUBE_STAGE') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    script {
                        sh """
                            mvn sonar:sonar \
                            -Dsonar.projectKey=i27-eureka-05 \
                            -Dsonar.host.url=http://34.86.250.120:9000 \
                            -Dsonar.login=${env.SONAR_TOKEN}
                        """
                    }
                }
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: true
                }
            }
            post {
                success {
                    script {
                        emailext(
                            subject: "Success: SonarQube Analysis for ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                            body: "SonarQube Analysis Passed.\n\nBuild Number: ${env.BUILD_NUMBER}\nJob URL: ${env.BUILD_URL}",
                            to: 'kishorecloud.1725@gmail.com'
                        )
                    }
                }
                failure {
                    script {
                        emailext(
                            subject: "Failure: SonarQube Analysis for ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                            body: "SonarQube Analysis Failed.\n\nBuild Number: ${env.BUILD_NUMBER}\nJob URL: ${env.BUILD_URL}",
                            to: 'kishorecloud.1725@gmail.com'
                        )
                    }
                }
            }
        }

        stage('DOCKER_BUILD_AND_PUSH') {
            steps {
                script {
                    sh "cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
                    sh "docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd"
                    sh "docker login -u ${env.DOCKER_CREDS_USR} -p ${env.DOCKER_CREDS_PSW}"
                    sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
                }
            }
            post {
                success {
                    script {
                        emailext(
                            subject: "Success: Docker Build and Push for ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                            body: "Docker build and push completed.\n\nBuild Number: ${env.BUILD_NUMBER}\nJob URL: ${env.BUILD_URL}",
                            to: 'kishorecloud.1725@gmail.com'
                        )
                    }
                }
                failure {
                    script {
                        emailext(
                            subject: "Failure: Docker Build and Push for ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                            body: "Docker build or push failed.\n\nBuild Number: ${env.BUILD_NUMBER}\nJob URL: ${env.BUILD_URL}",
                            to: 'kishorecloud.1725@gmail.com'
                        )
                    }
                }
            }
        }

        stage('DEPLOY_TO_DEV') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'john_docker_vm_creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        try {
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@'$DOCKER_VM' \"docker stop ${env.APPLICATION_NAME}-dev\""
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@'$DOCKER_VM' \"docker rm ${env.APPLICATION_NAME}-dev\""
                        }
                        catch (err) {
                            echo "Caught Error: $err"
                        }
                        sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@'$DOCKER_VM' \"docker container run -dit -p 8761:8761 --name ${env.APPLICATION_NAME}-dev ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}\""
                    }
                }
            }
            post {
                success {
                    script {
                        emailext(
                            subject: "Success: Deployment to Dev for ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                            body: "Deployment to Dev completed.\n\nBuild Number: ${env.BUILD_NUMBER}\nJob URL: ${env.BUILD_URL}",
                            to: 'kishorecloud.1725@gmail.com'
                        )
                    }
                }
                failure {
                    script {
                        emailext(
                            subject: "Failure: Deployment to Dev for ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                            body: "Deployment to Dev failed.\n\nBuild Number: ${env.BUILD_NUMBER}\nJob URL: ${env.BUILD_URL}",
                            to: 'kishorecloud.1725@gmail.com'
                        )
                    }
                }
            }
        }
    }
}
