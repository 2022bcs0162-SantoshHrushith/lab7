pipeline {
    agent any

    environment {
        IMAGE = "2022bcs0162santoshhrushith/red_wine_predict_jenkins_2022bcs0162:latest"
        CONTAINER_NAME = "wine_validation_test"
        PORT = "8001"
        BASE_URL = "http://host.docker.internal:8001"
    }

    stages {

        stage('Pull Image') {
            steps {
                sh "docker pull ${IMAGE}"
            }
        }

        stage('Run Container') {
            steps {
                sh """
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d -p ${PORT}:8000 --name ${CONTAINER_NAME} ${IMAGE}
                """
            }
        }

        stage('Wait for Service Readiness') {
            steps {
                script {
                    timeout(time: 90, unit: 'SECONDS') {
                        waitUntil {
                            def status = sh(
                                script: "curl -s -o /dev/null -w '%{http_code}' ${BASE_URL}/docs || true",
                                returnStdout: true
                            ).trim()

                            echo "HTTP Status: ${status}"
                            return status == "200"
                        }
                    }
                }
            }
        }

        stage('Send Valid Inference Request') {
            steps {
                script {
                    def response = sh(
                        script: """
                            curl -s -X POST ${BASE_URL}/predict \
                            -H "Content-Type: application/json" \
                            -d @tests/valid_input.json
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Valid Response: ${response}"

                    if (!response.contains("wine_quality")) {
                        error("Prediction field missing in valid response!")
                    }

                    if (!response.matches(".*\\d+.*")) {
                        error("Prediction value is not numeric!")
                    }
                }
            }
        }

        stage('Send Invalid Request') {
            steps {
                script {
                    def response = sh(
                        script: """
                            curl -s -X POST ${BASE_URL}/predict \
                            -H "Content-Type: application/json" \
                            -d @tests/invalid_input.json
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Invalid Response: ${response}"

                    if (!response.contains("detail")) {
                        error("Invalid input did not return proper error message!")
                    }
                }
            }
        }
    }

    post {
        always {
            sh """
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true
            """
        }
    }
}