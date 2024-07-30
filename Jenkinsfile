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
        DOCKER_REGISTRY_CREDENTIALS_ID = 'docker-auth'
        GITHUB_CREDENTIALS_ID = 'github-auth'
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

        stage('Update Commit Status') {
            steps {
                script {
                    def commitSha = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    def context = "Jenkins Build"
                    def description = "Build and push completed"
                    def state = "success"
                    def githubApiUrl = "https://api.github.com/repos/subhashis2018/jenkins-cicd-gradle/statuses/${commitSha}"

                    withCredentials([string(credentialsId: "${env.GITHUB_CREDENTIALS_ID}", variable: 'GITHUB_TOKEN')]) {
                        sh """
                            curl -H "Authorization: token ${GITHUB_TOKEN}" \
                                 -H "Content-Type: application/json" \
                                 -d '{"state": "${state}", "target_url": "${env.BUILD_URL}", "description": "${description}", "context": "${context}"}' \
                                 ${githubApiUrl}
                        """
                    }
                }
            }
        }
    }
}
