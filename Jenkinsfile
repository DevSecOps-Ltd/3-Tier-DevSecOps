pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '2'))
    }

    tools {
        nodejs 'nodejs'
    }

    stages {

        stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect --source . --exit-code 1 --redact'
            }
        }

        stage('Frontend Build') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                    sh 'CI=false npm run build'
                }
            }
        }

        stage('Backend Build') {
            steps {
                dir('backend') {
                    sh 'npm ci'
                }
            }
        }

        stage('SonarQube Analysis') {
           steps {
              withSonarQubeEnv('sonarqube') {
              sh '''
                sonar-scanner \
                  -Dsonar.projectKey=3-tier-DevSecOps \
                  -Dsonar.projectName=3-tier-DevSecOps \
                  -Dsonar.sources=frontend/src,backend
            '''
               }
            }
      }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    

    post {
        success {
            archiveArtifacts artifacts: 'frontend/build/**',
                             fingerprint: true
           }
       }
    }
}    