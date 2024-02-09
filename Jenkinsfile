pipeline {
    agent any

    environment {
        PROJECT_ID = "${PROJECT}"
        REGISTRY_REGION = 'us'
        REGISTRY_NAME = 'sftp-go'
        DOCKERFILE_PATH = 'path/to/Dockerfile'
        IMAGE_NAME = 'sftpgo'
        IMAGE_TAG = 'latest'
    }

    stages {
       stage('Build and push image with Container Builder') {
        steps {
          container('gcloud') {
              sh "PYTHONUNBUFFERED=1 gcloud builds submit -t ${REGISTRY_REGION}-docker.pkg.dev/${PROJECT}/${REGISTRY_NAME}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
          }
        }
       stage('Deploy to GKE') {
            steps {
               container('kubectl') {
                  step([$class: 'KubernetesEngineBuilder', namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/services', credentialsId: env.gcp-credentials, verifyDeployments: false])
                  step([$class: 'KubernetesEngineBuilder', namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/production', credentialsId: env.gcp-credentials, verifyDeployments: true])
                  sh("echo http://`kubectl --namespace=production get service/${IMAGE_NAME} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${IMAGE_NAME}")
                }
            }
       }

    post {
        success {
            echo 'Docker image built and pushed successfully to GCP Artifact Registry'
        }
        failure {
            echo 'Failed to build and push Docker image to GCP Artifact Registry'
        }
    }
}
