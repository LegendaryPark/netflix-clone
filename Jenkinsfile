pipeline{
    agent any

    tools{
        nodejs 'node16'
    }

    environment{
        SCANNER_HOME=tool 'sonar-scanner'
        APP_NAME = "netflix-clone"
        DOCKER_USER = "jpdylan89"
        DOCKER_PASS = "dockerhub"
        IMAGE_NAME = "${DOCKER_USER}"  + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages{
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/LegendaryPark/netflix-clone'
            }
        }
        stage('Sonarqube Analysis'){
            steps{
                withSonarQubeEnv('sonar-server'){
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Netflix \
                        -Dsonar.sources=. '''
                }
            }
        }
        stage('Quality Gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('OWASP FS SCAN'){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN'){
            steps{
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Build & Push Docker Image'){
            steps{
                script{
                    docker.withRegistry('', DOCKER_PASS){
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('', DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }
    }
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'pbjkmn00@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}