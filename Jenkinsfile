pipeline {

    agent { label 'server1' }

    tools {
        jdk 'java17'
        maven 'mvn3'
    }

    environment {
        SONAR_HOME = tool 'sonar'
    }

    parameters {
        string(name: 'DOCKER_TAG', defaultValue: '', description: 'Setting Docker image for latest push')
        string(name: 'PROJECT_NAME', defaultValue: '', description: 'Setting Docker image for latest push')
    }

    stages {

        stage('Validation Parameters') {
            steps {
                script {
                    if (params.DOCKER_TAG == '' || params.PROJECT_NAME == '') {
                        error("DOCKER_TAG and PROJECT_NAME must be provided")
                    }
                }
            }
        }

        stage('Workspace Cleanup') {
            steps {
                cleanWs()
            }
        }

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jaysh-rob/Boardgame.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy fs . -f table -o result.html"
            }
        }

        stage('SonarQube Scanner') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                        ${SONAR_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=Boardgame \
                        -Dsonar.projectKey=boardgame \
                        -Dsonar.java.binaries=target
                    """
                }
            }
        }

        stage('Package') {
            steps {
                sh "mvn package"
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t jackdhub/${params.PROJECT_NAME}:${params.DOCKER_TAG} ."
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker_cred',
                    usernameVariable: 'User_docker',
                    passwordVariable: 'Pass_docker'
                )]) {
                    script {
                        // More secure login
                        sh "echo $Pass_docker | docker login -u $User_docker --password-stdin"
                        sh "docker push $User_docker/${params.PROJECT_NAME}:${params.DOCKER_TAG}"
                    }
                }
            }
        }

        post{
        success{
            build job: "boardgame-cd", parameters: [
                string(name: 'DOCKER_TAG', value: "${params.DOCKER_TAG}"),
                string(name: 'PROJECT_NAME', value: "${params.PROJECT_NAME}")
            ]
            }
        }
    }
}
