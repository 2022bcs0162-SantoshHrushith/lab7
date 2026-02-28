pipeline {
    agent any

    environment {
        IMAGE = "2022bcs0162santoshhrushith/red_wine_predict_jenkins_2022bcs0162:latest"
        CONTAINER_NAME = "wine_validation_test"
        PORT = "8001"
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
                    docker run -d -p ${PORT}:8000 --name ${CONTAINER_NAME} ${IMAGE}
                """
            }
        }

        stage('Wait for Service Readiness') {
            steps {
                script {
                    timeout(time: 60, unit: 'SECONDS') {
                        waitUntil {
                            def status = sh(
                                script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${PORT}/docs || true",
                                returnStdout: true
                            ).trim()
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
                            curl -s -X POST http://localhost:${PORT}/predict \
                            -H "Content-Type: application/json" \
                            -d @tests/valid_input.json
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Valid Response: ${response}"

                    if (!response.contains("wine_quality")) {
                        error("Prediction field missing!")
                    }
                }
            }
        }

        stage('Send Invalid Request') {
            steps {
                script {
                    def response = sh(
                        script: """
                            curl -s -X POST http://localhost:${PORT}/predict \
                            -H "Content-Type: application/json" \
                            -d @tests/invalid_input.json
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Invalid Response: ${response}"

                    if (!response.contains("detail")) {
                        error("Invalid input did not produce error!")
                    }
                }
            }
        }

        stage('Stop Container') {
            steps {
                sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                """
            }
        }
    }
}