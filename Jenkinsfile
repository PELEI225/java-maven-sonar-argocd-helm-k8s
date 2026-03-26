pipeline {
    agent any

    tools {
        maven 'Maven 3.9.12'
        jdk 'JDK_17'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/PELEI225/java-maven-sonar-argocd-helm-k8s'
            }
        }

        stage('Build and Test') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('SonarQube Scan') {
            environment {
                SONAR_URL = "http://10.0.0.129:9000"
            }
            steps {
                withCredentials([string(credentialsId: 'sonar-token3', variable: 'SONAR_TOKEN')]) {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.host.url=$SONAR_URL \
                        -Dsonar.login=$SONAR_TOKEN
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t pelei225/ultimate-cicd:${BUILD_NUMBER} ."
            }
        }

        stage('DockerHub Login') {
            environment {
                DOCKERHUB_CREDENTIALS = credentials('DockerHub5')
            }
            steps {
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh "docker push pelei225/ultimate-cicd:${BUILD_NUMBER}"
            }
        }

        stage('Update Kubernetes Manifest') {
            environment {
                GIT_REPO_NAME = "java-maven-sonar-argocd-helm-k8s"
                GIT_USER_NAME = "PELEI225"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "ebenezerpelei7@gmail.com"
                        git config user.name "PELEI225"

                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml

                        git add spring-boot-app-manifests/deployment.yml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}" || echo "No changes to commit"

                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }

    post {
        success {
            slackSend(
                channel: '#test-jenkins',
                color: '#36a64f',
                message: "SUCCESS — Build #${BUILD_NUMBER} déployé avec succès !"
            )
        }
        failure {
            slackSend(
                channel: '#test-jenkins',
                color: '#ff0000',
                message: "FAILURE — Le pipeline #${BUILD_NUMBER} a échoué !"
            )
        }
    }
}
