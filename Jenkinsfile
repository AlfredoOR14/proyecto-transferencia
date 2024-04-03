pipeline {
    agent any

    environment {
        PROJECT_ID = 'devioz-pe-dev-analitica'
        NAME_SECRET = 'aws_Cred'
        GCP_SERVICE_ACCOUNT = 'devioz-corporativo-gcp-devops-analitica-dev'
        GCP_LOCATION = 'us-central1'
        NAME_BUCKET_GCP = 'mi-bucket-gcp-2'
        NAME_BUCKET_S3 = 'mi-bucket-aws-1'
        NAME_TRANSFER = 'PRUEBA'
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
        
        stage('Verificar y Actualizar Transferencia de Datos') {
            steps {
                script {
                    env.CLOUDSDK_CORE_DISABLE_PROMPTS = 'true'
                    def awsCredentials = sh(script: "gcloud secrets versions access latest --secret=${NAME_SECRET}", returnStdout: true).trim()
                    def awsCredentialsFilePath = "${env.WORKSPACE}/aws_credentials.json"
                    writeFile file: awsCredentialsFilePath, text: awsCredentials
                    
                    // Verificar si el trabajo de transferencia ya existe
                    def transferExists = sh(script: "gcloud transfer jobs describe ${NAME_TRANSFER}", returnStatus: true)
                    
                    // Si el trabajo de transferencia existe, eliminarlo antes de crear uno nuevo
                    if (transferExists == 0) {
                        sh "gcloud transfer jobs delete ${NAME_TRANSFER} --quiet"
                    }
                    
                    // Crear el nuevo trabajo de transferencia
                    sh """
                        gcloud transfer jobs create s3://${NAME_BUCKET_S3} gs://${NAME_BUCKET_GCP} \
                        --name=${NAME_TRANSFER} \
                        --source-creds-file=${awsCredentialsFilePath} \
                        --overwrite-when=different \
                        --schedule-repeats-every=1h \
                        --schedule-starts="2024-04-03T17:58:00Z"
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
