pipeline {
    agent any
    environment {
        DOCKER_USER = "2022bcs0088shashank"
        IMAGE_NAME = "${DOCKER_USER}/lab6:latest"
        CONTAINER_NAME = "inference-test-${env.BUILD_NUMBER}"
    }
    stages {
        stage('Pull Image') {
            steps {
                sh "docker pull ${IMAGE_NAME}"
            }
        }
        stage('Run Container') {
            steps {
                sh "docker run -d --name ${CONTAINER_NAME} -p 8000:8000 ${IMAGE_NAME}"
            }
        }
        stage('Wait for Service Readiness') {
            steps {
                sh "sleep 5" 
                sh "curl --retry 5 --retry-delay 2 http://localhost:8000/"
            }
        }
        stage('Send Valid Inference Request') {
            steps {
                script {
                    def response = sh(script: """
                        curl -s -X POST http://localhost:8000/predict \
                        -H 'Content-Type: application/json' \
                        -d '{"fixed_acidity": 7.4, "volatile_acidity": 0.7, "citric_acid": 0.0, "residual_sugar": 1.9, "chlorides": 0.076, "free_sulfur_dioxide": 11.0, "total_sulfur_dioxide": 34.0, "density": 0.9978, "pH": 3.51, "sulphates": 0.56, "alcohol": 9.4}'
                    """, returnStdout: true).trim()
                    
                    echo "Valid API Response: ${response}"
                    if (!response.contains("wine_quality")) {
                        error "Validation Failed: Prediction field missing!"
                    }
                }
            }
        }
        stage('Send Invalid Request') {
            steps {
                script {
                    def status = sh(script: "curl -s -o /dev/null -w '%{http_code}' -X POST http://localhost:8000/predict -H 'Content-Type: application/json' -d '{\"bad_field\": 0}'", returnStdout: true).trim()
                    
                    echo "Invalid Request Status Code: ${status}"
                    if (status != "422") {
                        error "Validation Failed: API did not return error for invalid input!"
                    }
                }
            }
        }
    }
    post {
        always {
            sh "docker stop ${CONTAINER_NAME} || true"
            sh "docker rm ${CONTAINER_NAME} || true"
        }
        success {
            echo "Pipeline Passed: All validations successful [cite: 454]"
        }
        failure {
            echo "Pipeline Failed: Validation checks did not pass [cite: 453]"
        }
    }
}