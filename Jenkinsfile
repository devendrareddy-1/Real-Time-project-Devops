pipeline {
    agent any

    environment {
        PROJECT_ID = "tough-study-474313-j2"        // 🔁 change if needed
        REGION     = "us-central1"
        REPO       = "devops-repo"
        IMAGE      = "gcp-devops-app"
        TAG        = "${BUILD_NUMBER}"
        GAR_PATH   = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE:$TAG .
                docker tag $IMAGE:$TAG $GAR_PATH:$TAG
                '''
            }
        }

        stage('Authenticate to GCP') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                    gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                    gcloud auth configure-docker $REGION-docker.pkg.dev --quiet
                    '''
                }
            }
        }

        stage('Push Image to Artifact Registry') {
            steps {
                sh '''
                docker push $GAR_PATH:$TAG
                '''
            }
        }

        stage('Deploy to GKE') {
            steps {
                sh '''
                kubectl apply -f k8s/
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
