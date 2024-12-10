pipeline {
    agent any
    
    environment {
        DOCKER_HUB_USERNAME = 'mbRabaa'
        DOCKER_HUB_REPO = 'mon_repo_docker'
        IMAGE_NAME = 'spring-boot-app'
        GITHUB_REPO = 'https://github.com/mbRabaa/examen-devops.git'
    }

    triggers {
        pollSCM('* * * * *')  // Scruter le dépôt toutes les 5 minutes
    }

    stages {
        stage('Cloner le code depuis GitHub') {
            steps {
                git url: "${GITHUB_REPO}", branch: 'master'
            }
        }

        stage('Build du projet') {
            steps {
                script {
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('Construire l\'image Docker') {
            steps {
                script {
                    writeFile file: 'Dockerfile', text: '''
                    FROM openjdk:17-jdk-slim
                    COPY target/*.jar app.jar
                    ENTRYPOINT ["java", "-jar", "/app.jar"]
                    '''
                    sh 'docker build -t ${DOCKER_HUB_USERNAME}/${DOCKER_HUB_REPO}:${IMAGE_NAME} .'
                }
            }
        }

        stage('Pousser l\'image sur Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }
                    sh 'docker push ${DOCKER_HUB_USERNAME}/${DOCKER_HUB_REPO}:${IMAGE_NAME}'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminé avec succès !'
        }

        failure {
            echo 'Une erreur s\'est produite durant le pipeline.'
        }
    }
}
