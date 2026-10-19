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

stage('Security Scan - Trivy') {
    steps {
        sh '''
            set -eu

            docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                -v trivy-cache:/root/.cache/ \
                aquasec/trivy:latest \
                image \
                --severity HIGH,CRITICAL \
                --exit-code 1 \
                "$IMAGE"
        '''
    }
}

stage('Smoke Test') {
    steps {
        sh '''
            set -eu

            docker run -d --name "$TEST_CONTAINER" "$IMAGE"

            ready=0
            for attempt in $(seq 1 15); do
                if docker exec "$TEST_CONTAINER" \
                    wget -q -O /dev/null http://127.0.0.1/; then
                    ready=1
                    break
                fi
                sleep 2
            done

            if [ "$ready" -ne 1 ]; then
                echo "Smoke test failed: Nginx did not become ready."
                docker logs "$TEST_CONTAINER" || true
                exit 1
            fi

            echo "Smoke test passed: Nginx served the page successfully."
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
