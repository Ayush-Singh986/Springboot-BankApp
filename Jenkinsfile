@Library('Shared') _
pipeline {
    agent any

    environment {
        SONAR_HOME = tool "Sonar" // Jenkins tool config name
    }

    parameters {
        string(name: 'DOCKER_TAG', defaultValue: '', description: 'Docker image tag for the build')
    }

    stages {
        stage("Workspace cleanup") {
            steps {
                cleanWs()
            }
        }

        stage("Git: Code Checkout") {
            steps {
                script {
                    code_checkout("https://github.com/Ayush-Singh986/Springboot-BankApp.git", "DevOps")
                }
            }
        }

        stage("Build: Compile Java") {
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
                withCredentials([string(credentialsId: 'sonar-token-id', variable: 'SONAR_TOKEN')]) {
                    sh """
                        ${SONAR_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=bankapp \
                        -Dsonar.projectKey=bankapp \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.login=$SONAR_TOKEN
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

        stage("Docker: Build Image") {
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
