pipeline {
    agent any
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'feature1', url: 'https://github.com/EmAdd9/Python-Demo-WebApp.git'
            }
        }
        stage('Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'owasp'
                     dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy FS') {
            steps {
                sh "trivy fs ."
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=py-webapp \
                    -Dsonar.projectKey=py-webapp '''
                }
            }
        }
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '017eb68a-96c0-4f3f-a3d4-ddaa78566ff3', toolName: 'docker') {
                        sh "make image"
                    }
                }
            }
        }
        stage('Trivy image-scan') {
            steps {
                sh "trivy image sudebdocker/python-webapp:latest"
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '017eb68a-96c0-4f3f-a3d4-ddaa78566ff3', toolName: 'docker') {
                        sh "make push"
                    }
                }
            }
        }
        stage('Deploy to container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '017eb68a-96c0-4f3f-a3d4-ddaa78566ff3', toolName: 'docker') {
                        sh "docker run -d --name py-app -p 5000:5000 sudebdocker/python-webapp:latest"
                    }
                }
            }
        }
    }
}
