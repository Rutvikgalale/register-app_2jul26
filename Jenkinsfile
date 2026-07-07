pipeline {
    agent { label "jenkins-agent" }

    tools {
        jdk "java17"
        maven "maven3"
    }

    environment {
        APP_NAME = "register_app"
        IMAGE_TAG = ""
    }

    stages {

        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: "main",
                    url: "https://github.com/Rutvikgalale/register-app_2jul26.git"
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Run Tests") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "docker",
                            usernameVariable: "DOCKER_USER",
                            passwordVariable: "DOCKER_PASS"
                        )
                    ]) {

                        env.IMAGE_TAG = "${DOCKER_USER}/${APP_NAME}:${BUILD_NUMBER}"

                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

                            docker build -t ${env.IMAGE_TAG} .

                            docker push ${env.IMAGE_TAG}

                            docker logout
                        """
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh """
                    docker run --rm \
                    -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy image \
                    ${env.IMAGE_TAG} \
                    --no-progress \
                    --scanners vuln \
                    --severity HIGH,CRITICAL \
                    --exit-code 0 \
                    --format table
                """
            }
        }

        stage("Deploy Container") {
            steps {
                sh """
                    docker rm -f ${APP_NAME} || true

                    docker run -d \
                    --name ${APP_NAME} \
                    -p 8080:8080 \
                    ${env.IMAGE_TAG}
                """
            }
        }
    }

    post {

        always {
            cleanWs()
        }

        success {
            echo "Pipeline completed successfully."
        }

        failure {
            echo "Pipeline failed."
        }
    }
}
