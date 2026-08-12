pipeline {
    agent { label 'Mindgate-Agent' }

    tools {
        jdk 'Java21'
        maven 'Maven3'
    }

    environment {
        IMAGE_NAME  = 'sagardaw/register-app-pipeline'
        IMAGE_TAG   = "${BUILD_NUMBER}"
        DOCKER_CRED = 'docker-credentials-id'
    }

    stages {
        stage("Cleanup Workspace & Disk Space") {
            steps {
                cleanWs()
                // Disk full cha issue avoid karnya sathi
                sh 'docker system prune -af --volumes || true'
                sh 'rm -rf ~/.cache/trivy || true'
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/milindsagar/register-app.git'
            }
        }

        stage("Build Application") {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage("Test Application") {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube-server') {
                        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar'
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    // Jenkins context madhil credentials use karnya sathi
                    withCredentials([usernamePassword(credentialsId: DOCKER_CRED, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push ${IMAGE_NAME}:${IMAGE_TAG}'
                        sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest'
                        sh 'docker push ${IMAGE_NAME}:latest'
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                script {
                    sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:${IMAGE_TAG} --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table"
                }
            }
        }

        stage('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
                    sh "docker rmi ${IMAGE_NAME}:latest || true"
                }
            }
        }

        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://3.110.172.242:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
            sh 'docker system prune -f || true'
        }
    }
}
