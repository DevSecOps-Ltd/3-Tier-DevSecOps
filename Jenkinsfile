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
        BACKEND_IMAGE  = "${ECR_REGISTRY}/${ECR_REPOSITORY}:backend-${IMAGE_TAG}"
    }

    stages {

        stage('Gitleaks Scan') {
            steps {
                sh '''
                    echo "Running Gitleaks scan..."
                    gitleaks detect \
                        --source . \
                        --exit-code 1 \
                        --redact
                '''
            }
        }

        stage('Frontend Build') {
            steps {
                dir('frontend') {
                    sh '''
                        echo "Installing frontend dependencies..."
                        npm ci

                        echo "Building frontend..."
                        CI=false npm run build
                    '''
                }
            }
        }

        stage('Backend Build') {
            steps {
                dir('backend') {
                    sh '''
                        echo "Installing backend dependencies..."
                        npm ci
                    '''
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

        stage('Docker Local Cleanup') {
            steps {
                sh '''
                    echo "Removing old local Docker images..."

                    docker rmi ${FRONTEND_IMAGE} || true
                    docker rmi ${BACKEND_IMAGE} || true

                    docker image prune -f
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "Building frontend Docker image..."

                    docker build \
                        -t ${FRONTEND_IMAGE} \
                        ./frontend

                    echo "Building backend Docker image..."

                    docker build \
                    -t ${BACKEND_IMAGE} \
                        ./backend

                    echo "Docker images created:"
                    docker images | grep three_tier_devsecops || true
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    echo "Scanning frontend image..."

                    trivy image \
                        --exit-code 1 \
                        --severity HIGH,CRITICAL \
                        ${FRONTEND_IMAGE}

                    echo "Scanning backend image..."

                    trivy image \
                        --exit-code 1 \
                        --severity HIGH,CRITICAL \
                        ${BACKEND_IMAGE}
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                    echo "Logging into Amazon ECR..."

                    aws sts get-caller-identity

                    aws ecr get-login-password \
                        --region ${AWS_REGION} |
                    docker login \
                        --username AWS \
                        --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Docker Push to ECR') {
            steps {
                sh '''
                    echo "Pushing frontend image..."

                    docker push ${FRONTEND_IMAGE}

                    echo "Pushing backend image..."

                    docker push ${BACKEND_IMAGE}
                '''
            }
        }

        stage('Docker Local Cleanup After Push') {
            steps {
                sh '''
                    echo "Cleaning local Docker images..."

                    docker rmi ${FRONTEND_IMAGE} || true
                    docker rmi ${BACKEND_IMAGE} || true

                    docker image prune -f
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs above.'
        }

        always {
            echo "Build Number: ${BUILD_NUMBER}"
            echo "Frontend Image: ${FRONTEND_IMAGE}"
            echo "Backend Image: ${BACKEND_IMAGE}"
        }
    }
}