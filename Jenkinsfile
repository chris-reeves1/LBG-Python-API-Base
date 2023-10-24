pipeline {
    agent any
    environment {
        GCR_CREDENTIALS_ID = 'gcp' // The ID you provided in Jenkins credentials
        IMAGE_NAME = 'test-image-5'
        GCR_URL = 'gcr.io/lbg-uplift-project'
        PROJECT_ID = 'lbg-uplift-project'
        CLUSTER_NAME = 'demo-cluster'
        LOCATION = 'europe-west2-c'
        CREDENTIALS_ID = 'lbg-uplift-project'
    }
    stages {
        stage('Build and Push to GCR') {
    steps {
        script {
            // Authenticate with Google Cloud
            withCredentials([file(credentialsId: GCR_CREDENTIALS_ID, variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
            }

            // Configure Docker to use gcloud as a credential helper
            sh 'gcloud auth configure-docker --quiet'

            // Build the Docker image
            sh "docker build -t ${GCR_URL}/${IMAGE_NAME}:${BUILD_NUMBER} ."

            // Push the Docker image to GCR
            sh "docker push ${GCR_URL}/${IMAGE_NAME}:${BUILD_NUMBER}"
        }
    }
        stage('deploy to GKE') {
    steps {
        script {
                    // Update the image in your deployment.yaml file
                    sh "sed -i 's|${GCR_URL}/${IMAGE_NAME}:latest|${GCR_URL}/${IMAGE_NAME}:${BUILD_NUMBER}|g' kubernetes/deployment.yaml"
                    
                    // Deploy to GKE using Jenkins Kubernetes Engine Plugin
                    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'kubernetes/deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
                }
            }
        }

        }
    }
}
