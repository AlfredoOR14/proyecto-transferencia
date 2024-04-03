pipeline {
    agent any

    environment {
        PROJECT_ID = 'HealthyLeaf'
        NAME_SECRET = 'aws_Cred'
        GCP_SERVICE_ACCOUNT = 'HealthyLeaf_'
        GCP_LOCATION = 'us-central1'
        NAME_BUCKET_GCP = 'mi-bucket-gcp-2'
        NAME_BUCKET_S3 = 'mi-bucket-aws-1'
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
                script {
                    sh "gcloud config set project ${PROJECT_ID}"
                    def bucketExists = sh(script: "gsutil ls gs://${NAME_BUCKET_GCP}", returnStatus: true)
                    if (bucketExists != 0) {
                        sh "gsutil mb -p ${PROJECT_ID} -l ${GCP_LOCATION} gs://${NAME_BUCKET_GCP}"
                    } else {
                        echo "El bucket gs://${NAME_BUCKET_GCP} ya existe. No se creará otro."
                    }
                }
            }
        }
        
         stage('Creacion de trasferencia de datos de AWS a GCP') {
            steps {
                script {
                    // Deshabilitar las solicitudes de activación de API
                    env.CLOUDSDK_CORE_DISABLE_PROMPTS = 'true'
                    // Recupera las credenciales de AWS desde Cloud Secret Manager
                    def awsCredentials = sh(script: "gcloud secrets versions access latest --secret=${NAME_SECRET}", returnStdout: true).trim()
                    // Ruta al archivo donde se guardarán las credenciales
                    def awsCredentialsFilePath = "${env.WORKSPACE}/aws_credentials.json"
                    // Escribir las credenciales en el archivo
                    writeFile file: awsCredentialsFilePath, text: awsCredentials
                    // Crea la transferencia de datos utilizando las credenciales recuperadas
                    sh """
                        gcloud transfer jobs create s3://${NAME_BUCKET_S3} gs://${NAME_BUCKET_GCP} \
                        --source-creds-file=${awsCredentialsFilePath} \
                        --overwrite-when=different \
                        --schedule-repeats-every=1h \
                        --schedule-starts="2024-04-02T17:58:00Z" 
                    """
                }
            }
        }

        stage('Limpiando Workspace') {
            steps {
                deleteDir()
            }
        }
    }
}
