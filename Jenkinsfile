pipeline {
    agent any

    environment {
        PROJECT_ID = 'devioz-pe-dev-analitica'
        NAME_SECRET = 'aws_Cred'
        GCP_SERVICE_ACCOUNT = 'devioz-corporativo-gcp-devops-analitica-dev'
        GCP_LOCATION = 'us-central1'
        NAME_BUCKET_GCP = 'mi-bucket-gcp-3'
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
                        echo "El bucket gs://${NAME_BUCKET_GCP} ya existe. No se creará otro."
                    } else {
                        sh "gsutil mb -p ${PROJECT_ID} -l ${GCP_LOCATION} gs://${NAME_BUCKET_GCP}"
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
                    // Autenticarse con AWS
                    sh "gcloud auth activate-service-account --key-file=${awsCredentialsFilePath}"
        
                    // Ejecutar el comando gsutil para copiar los datos de S3 a GCP
                    sh """
                        gsutil -o 'GSUtil:use_magicfile=True' cp -r s3://${NAME_BUCKET_S3} gs://${NAME_BUCKET_GCP}
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
