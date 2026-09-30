pipeline{
    agent any
    tools{
        nodejs 'nodejs'
    }
    stages{
        stage('Frontend Build'){
           steps{
              dir('frontend'){
                 sh 'npm ci'
                 sh 'npm run build'
                }
            }
        }
        stage('backend Build'){
           steps{
              dir('backend'){
                 sh 'npm ci'
                 sh 'npm run build'
                }
            }
       }
    }
}    