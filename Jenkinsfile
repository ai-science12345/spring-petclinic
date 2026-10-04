  GNU nano 7.2                                                                                               Jenkinsfile
































pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '986918902913'
        ECR_REPO = 'petclinic'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
        EKS_CLUSTER = 'devops-project2'
        K8S_NAMESPACE = 'petclinic'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Unit Test') {
            steps {
                sh './mvnw test'
            }
        }

        stage('Build Image') {
            steps {
                sh """
                    ./mvnw spring-boot:build-image \
                      -Dspring-boot.build-image.imageName=${IMAGE_URI}
                """
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                    trivy image \
                      --severity HIGH,CRITICAL \
                      --exit-code 1 \
                      ${IMAGE_URI}
                """
            }
        }

        stage('Push ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region ${AWS_REGION} |
                    docker login --username AWS --password-stdin \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                    docker push ${IMAGE_URI}
                """
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    kubectl set image deployment/petclinic \
                      petclinic=${IMAGE_URI} \
                      -n ${K8S_NAMESPACE}

                    kubectl rollout status deployment/petclinic \
                      -n ${K8S_NAMESPACE} \
                      --timeout=180s
                """
            }
        }

        stage('Verify') {
            steps {
                sh """
                    kubectl get pods -n ${K8S_NAMESPACE} -o wide
                    kubectl get svc petclinic -n ${K8S_NAMESPACE}
                """
            }
        }
    }

    post {
        success {
            echo 'PetClinic CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'PetClinic CI/CD pipeline failed.'
        }
    }
}






















                                                        
