pipeline {
    agent any
    environment {
        DOCKER_REPO_NAME = "dangeorge23/java-app"
        GITHUB_USERNAME = "dangeorge"
        GITHUB_REPO = "JavaApp-using-JenkinsCICD-on-Kubernetes"
        GITHUB_EMAIL = "dangeorge13@gmail.com"
    }

    stages {
        stage('Git Clone') {
            steps {
                echo 'Cloning Git repository'
                git branch: 'main', url: 'https://github.com/${GITHUB_USERNAME}/${GITHUB_REPO}.git'
            }
        }
        stage('Build Jar file') {
            steps {
                echo 'Building Jar File'
                sh 'mvn clean package'
            }
        }
        stage('Static Code Analysis') {
            steps {
                echo 'Analysing'
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=http://100.0.0.10:9000'
                }
            }
        }
        stage('Docker build') {
            steps {
                script {
                    echo 'Buidling docker image'
                    dockerImage = docker.build("${DOCKER_REPO_NAME}:build-${BUILD_NUMBER}")
                }
            }
        }
        stage('Push Docker image to Docker Hub') {
            steps {
                script {
                    echo 'Pushing docker image to Docker Hub'
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhublogin') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Cleaning Up') {
            steps {
                echo 'Removing docker image to free up space in the server'
                sh 'docker rmi ${DOCKER_REPO_NAME}:build-${BUILD_NUMBER}'
            }
        }
        stage('Update Deployment File') {
            steps {
                echo 'Updating deployment file with the build number'
                withCredentials([string(credentialsId: 'githublogin', variable: 'GITHUB_TOKEN')]) {
                sh '''
                git config user.email "${GITHUB_EMAIL}"
                git config user.name "${GITHUB_USERNAME}"
                git remote set-url origin https://${GITHUB_USERNAME}:${GITHUB_TOKEN}@github.com/${GITHUB_USERNAME}/${GITHUB_REPO}.git
                sed -i "/java-app:build-/s/build-.*/build-${BUILD_NUMBER}/" manifests/deployment.yaml
                git add manifests/deployment.yaml
                git commit -m "Updated build number to ${BUILD_NUMBER} in manifests/deployment.yaml file"
                git push
                '''
                }
            }
        }
    }
}
