pipeline {
    agent any
    environment {
        DOCKER_REPO_NAME = "dangeorge23/java-app"
    }

    stages {
        stage('Git Clone') {
            steps {
                echo 'Cloning Git repository'
                git branch: 'main', url: 'https://github.com/dangeorge/JavaApp-using-JenkinsCICD-on-Kubernetes.git'
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
                git config user.email "dangeorge13@gmail.com"
                git config user.name "dangeorge"
                git remote set-url origin https://dangeorge:${GITHUB_TOKEN}@github.com/dangeorge/JavaApp-using-JenkinsCICD-on-Kubernetes.git
                sed -i "/java-app:build-/s/build-.*/build-${BUILD_NUMBER}/" manifests/deployment.yaml
                git add manifests/deployment.yaml
                git commit -m "Updated build number to ${BUILD_NUMBER} in manifests/deployment.yaml file"
                git push --set-upstream origin main
                '''
                }
            }
        }
    }
}
