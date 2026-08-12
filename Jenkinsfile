pipeline {
    agent { label 'Mindgate-Agent' }
    tools {
        jdk 'Java21'
        maven 'Maven3'
    }
    
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId:'github', url: 'https://github.com/milindsagar/register-app.git'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }
        }
        stage("Test Application"){
            steps {
                sh "mvn test"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'Mindgate-Token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
    }
}
