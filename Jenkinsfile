pipeline {
    agent { label 'Mindgate-Agent' }

    tools {
        maven 'Maven3'
        jdk 'JDK21'
    }

    stages {
        stage('Cleanup Disk & Cache') {
            steps {
                // Pre-build disk space clear karnyasathi
                sh 'docker system prune -af --volumes || true'
                sh 'rm -rf ~/.cache/trivy || true'
                cleanWs()
            }
        }

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test Application') {
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

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                        sh 'docker build -t sagardaw/register-app-pipeline:${BUILD_NUMBER} .'
                        sh 'docker push sagardaw/register-app-pipeline:${BUILD_NUMBER}'
                        sh 'docker tag sagardaw/register-app-pipeline:${BUILD_NUMBER} sagardaw/register-app-pipeline:latest'
                        sh 'docker push sagardaw/register-app-pipeline:latest'
                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    sh '''
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy image sagardaw/register-app-pipeline:${BUILD_NUMBER} \
                        --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table
                    '''
                }
            }
        }

        stage('Cleanup Artifacts') {
            steps {
                sh 'docker image prune -f || true'
            }
        }
    }

    post {
        always {
            // Unused space parat clear honyasathi
            cleanWs()
            sh 'docker system prune -f || true'
        }
    }
}
