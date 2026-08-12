pipeline {
    agent { label 'Mindgate-Agent' }

    tools {
        jdk 'Java21'
        maven 'Maven3'
    }

    environment {
        // Docker Image details आणि Credentials चा वापर
        IMAGE_NAME  = 'sagardaw/register-app-pipeline'
        IMAGE_TAG   = "${BUILD_NUMBER}"
        DOCKER_PASS = 'docker-credentials-id' // Jenkins मधील Dockerhub Credentials ID
    }

    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/milindsagar/register-app.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Jenkins Global Configuration मधील Sonar Server चे नाव
                    withSonarQubeEnv('sonarqube-server') {
                        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar'
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    // waitForQualityGate ला credentialsId लागत नाही
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', DOCKER_PASS) {
                        def docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                script {
                    sh "docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:${IMAGE_TAG} --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table"
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
    }
}
