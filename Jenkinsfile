pipeline{
    agent any
    tools{
        nodejs 'nodejs'
    }
    stages{
        stage('compile'){
            steps{
                dir('frontend'){
                    sh 'find . -name "*.js" -exec node {} +'
                }
            }
        }
        stage('compile again'){
            steps{
                dir('backend'){
                    sh 'find . -name "*.js" -exec node {} +'
                }
            }
        }
    }
}