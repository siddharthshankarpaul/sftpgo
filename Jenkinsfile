pipeline {

    environment {
        PROJECT_ID = "${PROJECT}"
        REGISTRY_REGION = 'us'
        REGISTRY_NAME = 'sftp-go'
        IMAGE_NAME = 'sftpgo'
        IMAGE_TAG = 'latest'
    }
   agent {
    kubernetes {
       label "sftpgo-app"
       yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: gcloud
                image: google/cloud-sdk:latest
                command:
                - cat
                tty: true
              - name: kubectl
                image: lachlanevenson/k8s-kubectl:v1.21.2
                command:
                - cat
                tty: true
       """  
    }
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
                    // kubernetesEngineBuilder(namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/services', credentialsId: env.gcp_credentials, verifyDeployments: false)
                    // kubernetesEngineBuilder(namespace:'production', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: 'k8s/production', credentialsId: env.gcp_credentials, verifyDeployments: true)
                    // sh("echo http://`kubectl --namespace=production get service/${IMAGE_NAME} -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` > ${IMAGE_NAME}")
                }
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
