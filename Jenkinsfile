pipeline {
    agent any

    environment {
        IMAGE = "sre-portfolio:${BUILD_NUMBER}"
        TEST_CONTAINER = "sre-portfolio-test-${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    set -eu
                    test -f Dockerfile
                    test -f index.html
                    echo "Required project files found."
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t "$IMAGE" .'
            }
        }

        stage('Smoke Test') {
            steps {
                sh '''
                    set -eu
                    docker run -d --name "$TEST_CONTAINER" "$IMAGE"

                    docker exec "$TEST_CONTAINER" \
                        wget -q -O /dev/null http://127.0.0.1/

                    echo "Smoke test passed: Nginx returned a successful response."
                '''
            }
        }
    }

    post {
        always {
            sh '''
                docker rm -f "$TEST_CONTAINER" 2>/dev/null || true
            '''
        }

        success {
            echo 'CI pipeline completed successfully.'
        }

        failure {
            echo 'CI pipeline failed. Review the stage logs.'
        }
    }
}
