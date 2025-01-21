pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        
        stage('gitCheckout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/SalmanSk7/BoardGame.git'
            }
        }

        stage('complileTheCode') {
            steps {
                sh "mvn compile"
            }
        }

        stage('testCases') {
            steps {
                sh "mvn test"
            }
        }

        stage('fileSystemScanTrivy') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }

        stage('sonarQubeAnanlysis') {
            steps {
                withSonarQubeEnv('sonar-boardgame') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame \
                    -Dsonar.projectKey=BoardGame -Dsonar.java.binaries=.'''
                }
            }
        }

        stage('Wait for 5 seconds') {
            steps {
                sh 'sleep 5'
            }
        }

        stage('qualityGate') {
            steps {
                script {
                   waitForQualityGate abortPipeline: false, credentialsId: 'sonar-boardgame'
                }
            }
        }

        stage('buildStage') {
            steps {
                sh 'mvn package'
            }
        }

        stage('buildTagDockerImage') {
            steps {
                withDockerRegistry(credentialsId: 'docker', url: 'https://index.docker.io/v1/') {
                    sh "docker build -t salmansk15/boardgamesalman:dev ."
                }
            }
        }

        stage('dockerImageScan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html salmansk15/boardgamesalman:dev"
            }
        }
    }
}
