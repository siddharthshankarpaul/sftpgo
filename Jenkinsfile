pipeline {
    environment {
        PROJECT = "gke-proj-2904"
        REGISTRY_REGION = 'us'
        REGISTRY_NAME = 'sftpgo'
        IMAGE_NAME = 'sftpgo'
        IMAGE_TAG = 'latest'
        CLUSTER ='terraform-gke-cluster'
        CLUSTER_ZONE = 'us-west1'
    }

    agent {
        kubernetes {
            label 'sftpgo-app'
            yaml """
                apiVersion: v1
                kind: Pod
                serviceAccountName: terraform
                spec:
                  containers:
                  - name: gcloud
                    image: google/cloud-sdk:latest
                    command:
                    - cat
                    tty: true
                  - name: kubectl
                    image: gcr.io/cloud-builders/kubectl
                    command:
                    - cat
                    tty: true    
            """
        }
    }

    stages {
        stage('Build and push image') {
            steps {
                container('gcloud') {
                    script {
                        withCredentials([file(credentialsId: 'gke_sa', variable: 'GC_KEY')]) {
                            sh("gcloud auth activate-service-account --key-file=${GC_KEY}")
                            sh "PYTHONUNBUFFERED=1 gcloud builds submit -t ${REGISTRY_REGION}-docker.pkg.dev/${PROJECT}/${REGISTRY_NAME}/${IMAGE_NAME}:${IMAGE_TAG} ."
                        }
                    }
                }
            }
        }
         stage('Deploy to GKE') {
            steps {
                container('kubectl') {
                    withCredentials([file(credentialsId: 'gke_sa', variable: 'GC_KEY')]) {
                        sh("gcloud auth activate-service-account --key-file=${GC_KEY}")
                        sh "gcloud container clusters get-credentials ${CLUSTER} --zone ${CLUSTER_ZONE} --project ${PROJECT}"
                        sh "kubectl create deployment sftpgo-deployment --image=${REGISTRY_REGION}-docker.pkg.dev/${PROJECT}/${REGISTRY_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "kubectl set image deployment/sftpgo-deployment ${IMAGE_NAME}=${REGISTRY_REGION}-docker.pkg.dev/${PROJECT}/${REGISTRY_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "kubectl expose deployment sftpgo-deployment --port=2022 --port=8080 --type=LoadBalancer --name=sftpgo-service"
                    }
                }
            }
        }
        
    }
}
