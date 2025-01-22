pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        NEW_IMAGE_NAME = "salmansk15/boardgamesalman:main"
        GIT_USER_NAME = credentials('git-username')
        GITHUB_TOKEN = credentials('github-token')
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
                    sh "docker build -t $NEW_IMAGE_NAME ."
                }
            }
        }

        stage('dockerImageScan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html $NEW_IMAGE_NAME"
            }
        }

        stage('pushDockerImage') {
            steps {
                withDockerRegistry(credentialsId: 'docker', url: 'https://index.docker.io/v1/') {
                    sh "docker push $NEW_IMAGE_NAME"
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/SalmanSk7/BoardGame.git'
            }
        }

        stage('Update Deployment File') {
            steps {
                sh "sed -i 's|image: .*|image: $NEW_IMAGE_NAME|' argocd/01-deployment.yaml"
                sh "cat argocd/01-deployment.yaml"
                sh "git config user.name 'Jenkins CI'"
                sh "git config user.email 'jenkins@example.com'"
                sh "git config --global --add safe.directory ./"
                sh 'git add argocd/01-deployment.yaml'
                sh "git commit -m 'Update deployment image to $NEW_IMAGE_NAME'"
                sh "git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/BoardGame HEAD:main"

            }
        }
    }
}