pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '2'))
    }

    tools {
        nodejs 'nodejs'
    }

    environment {
        AWS_REGION = 'us-east-1'

        ECR_REGISTRY = '463556655164.dkr.ecr.us-east-1.amazonaws.com'

        ECR_REPOSITORY = 'three_tier_devsecops'

        IMAGE_TAG = "${BUILD_NUMBER}"

        FRONTEND_IMAGE = "${ECR_REGISTRY}/${ECR_REPOSITORY}:frontend-${IMAGE_TAG}"

        BACKEND_IMAGE = "${ECR_REGISTRY}/${ECR_REPOSITORY}:backend-${IMAGE_TAG}"
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
                 script {
                    def scannerHome = tool 'sonar'

                withEnv(["PATH+SONAR=${scannerHome}/bin"]) {
                    sh '''
                        sonar-scanner \
                            -Dsonar.projectKey=3-tier-DevSecOps \
                            -Dsonar.projectName=3-tier-DevSecOps \
                            -Dsonar.sources=frontend/src,backend
                    '''
                        }
                    }
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
    
        stage('docker build') {
            steps {
                sh  '''
                     docker build --no-cache -t${FRONTEND_IMAGE} ./frontend
                     docker build --no-cache -t ${BACKEND_IMAGE} ./backend
                      
                    '''
                  
                }
            }

         stage('trivy scan') {
            steps {
                sh  '''
                       trivy image --exit-code 1 --severity HIGH,CRITICAL ${FRONTEND_IMAGE}
                       trivy image --exit-code 1 --severity HIGH,CRITICAL ${BACKEND_IMAGE}
                    '''
                }
            }   
        }
    }
          