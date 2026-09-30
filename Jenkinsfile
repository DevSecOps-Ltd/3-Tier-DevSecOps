pipeline{
    agent any
    tools{
        nodejs 'nodejs'
    }
    stages{
        stage('Gitleaks Scan') {
           steps {
              sh 'gitleaks detect --source . --exit-code 1 --redact'
            }
       }
        stage('Frontend Build'){
           steps{
              dir('frontend'){
                 sh 'npm ci'
                 sh 'CI=false npm run build'
                }
            }
        }
        stage('backend Build'){
           steps{
              dir('backend'){
                 sh 'npm ci'
                }
            }
       }
    }
}    