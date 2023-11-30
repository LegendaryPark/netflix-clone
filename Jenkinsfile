pipeline{
    agent any

    tools{
        nodejs 'node16'
    }

    environment{
        SCANNER_HOME=tool 'sonar-scanner'
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