pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DevArun1/Multi-Tier-With-Database.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('test') {
            steps {
                sh "mvn test -DskipTest=true"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                        sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Multitier -Dsonar.projectKey=Multitier -Dsonar.java.binaries=target"
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn package -DskipTest=true"
            }
        }

        stage('Publish To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTest=true"
                }
            }
        }

        stage('Docker Build Image ') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t arun/bankapp.latest ."
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html arun/bankapp.latest"
            }
        }

         stage('Docker Push Image ') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push -t arun/bankapp.latest ."
                    }
                }
            }
        }

        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
     }
}
