pipeline {
    agent any
    environment {
        scannerHome = tool 'mysonar'
    }

    stages {
        stage('Cleanws') {
            steps {
                cleanWs()
            }
        }
        stage('Code') {
            steps {
                git 'https://github.com/kiranharsha9114-dot/ltibbhackathon-project-kubernetes.git'
            }
        }
        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh "${scannerHome}/bin/sonar-scanner -D sonar.projectKey=Blood-Bank"
                    
                }
            }
        }
        stage('Quality Gates') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-id'
                }
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t appimage .'
                sh 'docker build -t dbimage database/'
            }
        }
        stage('Scan Images') {
            steps {
                sh 'trivy image appimage --severity CRITICAL,HIGH > appimage-vul.txt'
                sh 'trivy image dbimage --severity CRITICAL,HIGH > dbimage-vul.txt'
            }
        }
        stage('Tag&Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-id') {
                        sh 'docker tag dbimage kiranreddy678/blood-bank:dbimage'
                        sh 'docker tag appimage kiranreddy678/blood-bank:appimage'
                        sh 'docker push kiranreddy678/blood-bank:dbimage'
                        sh 'docker push kiranreddy678/blood-bank:appimage'
                        
                    }
                }
            }
        }
    }
    post {
        always {
            mail to: 'chinnareddy9114@gmail.com',
            subject: 'Pipeline Status',
            body: "${currentBuild.currentResult}: ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at : ${env.BUILD_URL}"
        }
    }
}
