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
        DOCKER_IMAGE = "subhashis2022/jenkins-cicd-gradle"
        DOCKER_TAG = "${params.ENV}-${env.BUILD_ID}"
        DOCKER_REGISTRY_CREDENTIALS_ID = 'docker_auth'
        GITHUB_CREDENTIALS_ID = 'github_auth'
        GITHUB_REPO = 'https://github.com/subhashis2018/jenkins-cicd-gradle.git'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    git url: "${env.GITHUB_REPO}", credentialsId: "${env.GITHUB_CREDENTIALS_ID}"
                }
            }
        }

        stage('Prepare') {
            steps {
                script {
                    sh 'chmod +x ./gradlew'
                }
            }
        }

        stage('Build and Test') {
            steps {
                script {
                    sh './gradlew clean build'
                }
            }
        }

        stage('Run PMD and JaCoCo Reports') {
            steps {
                script {
                    sh './gradlew pmdMain pmdTest'
                    sh './gradlew jacocoTestReport'
                }
            }
            post {
                always {
                    publishHTML(target: [reportDir: 'build/reports/pmd', reportFiles: 'main.html', reportName: 'PMD Report'])
                    publishHTML(target: [reportDir: 'build/reports/jacoco/test/html', reportFiles: 'index.html', reportName: 'JaCoCo Report'])
                    jacoco(execPattern: 'build/jacoco/test.exec', sourcePattern: 'src/main/java', classPattern: 'build/classes/java/main', exclusionPattern: '')
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_REGISTRY_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }
    }
}
