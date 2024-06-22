pipeline {
    agent any

    parameters {
        choice(name: 'ENV', choices: ['dev', 'staging', 'production'], description: 'Choose the environment to deploy')
    }

    tools {
        gradle 'gradle-8.5'
        jdk 'jdk-17'
        git 'git'
    }

    environment {
        DOCKER_IMAGE = "myapp/jenkins-cicd-gradle"
        DOCKER_TAG = "${params.ENV}-${env.BUILD_ID}"
        DOCKER_REGISTRY_CREDENTIALS_ID = 'docker_auth'
        DOCKER_REGISTRY_USER='subhashis2022'
        GITHUB_CREDENTIALS_ID = 'github_auth'
        GITHUB_REPO = 'https://github.com/subhashis2018/jenkins-cicd-gradle.git'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the repository from GitHub
                    git url: "${env.GITHUB_REPO}", credentialsId: "${env.GITHUB_CREDENTIALS_ID}"
                }
            }
        }

        stage('Prepare') {
            steps {
                script {
                    // Change permissions of gradlew
                    sh 'chmod +x ./gradlew'
                }
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    // Build and test the application
                    sh './gradlew clean build'
                }
            }
        }

        stage('Run PMD and JaCoCo Reports') {
            steps {
                script {
                    // Generate PMD and JaCoCo reports
                    sh './gradlew pmdMain pmdTest'
                    sh './gradlew jacocoTestReport'
                }
            }
            post {
                always {
                    // Archive PMD and JaCoCo reports
                    publishHTML(target: [reportDir: 'build/reports/pmd', reportFiles: 'main.html', reportName: 'PMD Report'])
                    publishHTML(target: [reportDir: 'build/reports/jacoco/test/html', reportFiles: 'index.html', reportName: 'JaCoCo Report'])
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("${DOCKER_REGISTRY_USER}", "${DOCKER_REGISTRY_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }
    }
}
