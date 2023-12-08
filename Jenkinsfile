pipeline {
    agent any 
    
    environment {
        GCLOUD_PROJECT = 'csye7125-gke-f8c10e6f'
        GCLOUD_CLUSTER = 'my-gke-cluster'
        GCLOUD_REGION = 'us-east1'
        ISTIO_NAMESPACE = 'istio-system'
    }
    
    stages {
        stage('Authenticating with GKE') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'gke_service_account_key', variable: 'GCLOUD_SERVICE_KEY')]) {
                        sh "gcloud auth activate-service-account --key-file=${GCLOUD_SERVICE_KEY}"
                    }

                    sh "gcloud config set project ${GCLOUD_PROJECT}"
                    sh "gcloud container clusters get-credentials my-gke-cluster --region ${GCLOUD_REGION}"

                    echo "Authenticated with GKE..."

                }
            }
        }

        stage('Checking out the code') {
            steps{
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: 'main']],
                        userRemoteConfigs: [[credentialsId: 'github_token', url: 'https://github.com/csye7125-fall2023-group07/istio-helm-chart.git']]
                    ])
                }
            }
        }

        stage('Check and Create Namespace') {
            steps {
                script {
                    def namespaceExists = sh(script: "kubectl get namespace ${ISTIO_NAMESPACE}", returnStatus: true)
                    
                    if (namespaceExists != 0) {
                        sh "kubectl create namespace ${ISTIO_NAMESPACE}"
                        echo "Namespace ${ISTIO_NAMESPACE} created."
                    } else {
                        echo "Namespace ${ISTIO_NAMESPACE} already exists."
                    }
                }
            }
        }

        stage('Install Istio') {
            steps {
                sh 'helm repo add istio https://istio-release.storage.googleapis.com/charts'
                sh 'helm repo update'
                sh 'helm install istio-base istio/base -n ${ISTIO_NAMESPACE}'
                sh 'helm install istiod istio/istiod -n ${ISTIO_NAMESPACE} --wait'
                sh 'helm ls -n ${ISTIO_NAMESPACE}'
                sh 'kubectl get deployments -n ${ISTIO_NAMESPACE} --output wide'
                sh 'helm install istio-ingress istio/gateway -n ${ISTIO_NAMESPACE} --wait'
            }
        }
    }
}