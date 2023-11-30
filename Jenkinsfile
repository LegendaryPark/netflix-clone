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

    }
}