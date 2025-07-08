@Library('Shared') _
pipeline {
    agent any

    environment {
        SONAR_HOME = tool "Sonar"
    }

    parameters {
        string(name: 'DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    }

    stages {

        stage("Workspace cleanup") {
            steps {
                script {
                    cleanWs()
                }
            }
        }

        stage('Git: Code Checkout') {
            steps {
                script {
                    code_checkout("https://github.com/Ayush-Singh986/Springboot-BankApp.git", "DevOps")
                }
            }
        }

        stage("Build: Java Compile") {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage("Trivy: Filesystem scan") {
            steps {
                script {
                    trivy_scan()
                }
            }
        }

        stage("OWASP: Dependency check") {
            steps {
                script {
                    owasp_dependency()
                }
            }
        }

        stage("SonarQube: Code Analysis") {
            steps {
                script {
                    // Updated sonar-scanner call with sonar.java.binaries
                    sh """
                        ${SONAR_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=bankapp \
                        -Dsonar.projectKey=bankapp \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        stage("SonarQube: Code Quality Gates") {
            steps {
                script {
                    sonarqube_code_quality()
                }
            }
        }

        stage("Docker: Build Images") {
            steps {
                script {
                    docker_build("bankapp", "${params.DOCKER_TAG}", "ayush244")
                }
            }
        }

        stage("Docker: Push to DockerHub") {
            steps {
                script {
                    docker_push("bankapp", "${params.DOCKER_TAG}", "ayush244")
                }
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: '*.xml', followSymlinks: false
            build job: "BankApp-CD", parameters: [
                string(name: 'DOCKER_TAG', value: "${params.DOCKER_TAG}")
            ]
        }
    }
}
