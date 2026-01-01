pipeline {
    agent any

    environment {
        PROJECT_ID = "tough-study-474313-j2"
        REGION     = "us-central1"
        REPO       = "devops-repo"
        IMAGE      = "gcp-devops-app"
        TAG        = "${BUILD_NUMBER}"
        GAR_PATH   = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                bat '''
                docker build -t %IMAGE%:%TAG% .
                docker tag %IMAGE%:%TAG% %GAR_PATH%:%TAG%
                '''
            }
        }

        stage('Authenticate to GCP') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GCP_KEY')]) {
                    bat '''
                    gcloud auth activate-service-account --key-file=%GCP_KEY%
                    gcloud auth configure-docker %REGION%-docker.pkg.dev --quiet
                    '''
                }
            }
        }

        stage('Push Image to Artifact Registry') {
            steps {
                bat '''
                docker push %GAR_PATH%:%TAG%
                '''
            }
        }

        stage('Deploy to GKE') {
            steps {
                bat '''
                kubectl apply -f k8s
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
