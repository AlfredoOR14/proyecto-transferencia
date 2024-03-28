pipeline {
    agent any

    environment {
        PROJECT_ID = 'devioz-pe-dev-analitica'
        ENVIRONMENT = 'dev'
        GCP_SERVICE_ACCOUNT = 'devioz-corporativo-gcp-devops-analitica-dev'
        GCP_LOCATION = 'us-central1'
        NAME_BUCKET_GCP = 'mi-bucket2'
        NAME_BUCKET_S3 = 'alfredo02711'
    }
    stages {
        stage('Descarga de Fuentes') {
            steps {
                script {
                    deleteDir()
                    checkout scm
                }
            }
        }

        stage('Activando Service Account') {
            steps {
                withCredentials([file(credentialsId: "${GCP_SERVICE_ACCOUNT}", variable: 'SECRET_FILE')]) {
                    sh """\$(gcloud auth activate-service-account --key-file=\$SECRET_FILE)"""
                }
            }
        }
        
        stage('Create Bucket in GCP') {
            steps {
                sh "gcloud config set project ${PROJECT_ID}"
                sh "gsutil mb -p ${PROJECT_ID} -l ${GCP_LOCATION} gs://${NAME_BUCKET_GCP}"
            }
        }
        
        stage('Creacion de trasferencia de datos de AWS a GCP') {
            steps {
                sh "gcloud transfer jobs create \
                    s3://${NAME_BUCKET_S3} gs://${NAME_BUCKET_GCP} \
                    --source-auth-method=AWS_SIGNATURE_V4 \
                    --include-modified-after-relative=1d \
                    --schedule-repeats-every=1d \
                    --aws-access-key-id=${AWS_ACCESS_KEY_ID} \
                    --aws-secret-access-key=${AWS_SECRET_ACCESS_KEY}"
            }
        }
    }
}
