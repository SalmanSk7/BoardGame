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

        stage('publishArtifactToNexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }

        stage('buildTagDockerImage') {
            steps {
                withDockerRegistry(credentialsId: 'docker', url: 'https://index.docker.io/v1/') {
                    sh "docker build -t salmansk15/boardgamesalman:latest ."
                }
            }
        }

        stage('dockerImageScan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html salmansk15/boardgamesalman:latest"
            }
        }

        stage('pushDockerImage') {
            steps {
                withDockerRegistry(credentialsId: 'docker', url: 'https://index.docker.io/v1/') {
                    sh "docker push salmansk15/boardgamesalman:latest"
                }
            }
        }

        stage('kubernetesDeployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'minikube', credentialsId: 'k8s-cred', namespace: 'webapps', serverUrl: 'https://192.168.49.2:8443') {
                    sh "kubectl apply -f deployment-service.yaml"
                }
            }
        }

        stage('verifyingDeployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'minikube', credentialsId: 'k8s-cred', namespace: 'webapps', serverUrl: 'https://192.168.49.2:8443') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }

    post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

                def body = """
                    <html>
                    <body>
                    <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                    <h2>${jobName} - Build ${buildNumber}</h2>
                    <div style="background-color: ${bannerColor}; padding: 10px;">
                    <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                    </div>
                    <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                    </div>
                    </body>
                    </html>
                """

                emailext(
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                    body: body,
                    to: 'sample@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-image-report.html'
                )
            }
        }
    }
}
