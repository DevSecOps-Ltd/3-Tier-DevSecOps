pipeline{
    agent any
    tools{
        nodejs 'nodejs'
    }
    stages{
        stage('compile'){
            steps{
                sh 'npm run build'
            }
        }
    }
}