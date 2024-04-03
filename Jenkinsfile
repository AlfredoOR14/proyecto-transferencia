pipeline {
    agent any

    environment {
        PROJECT_ID = 'devioz-pe-dev-analitica'
        NAME_SECRET = 'aws_Cred'
        GCP_SERVICE_ACCOUNT = 'devioz-corporativo-gcp-devops-analitica-dev'
        GCP_LOCATION = 'us-central1'
        NAME_BUCKET_GCP = 'mi-bucket-gcp-2'
        NAME_BUCKET_S3 = 'mi-bucket-aws-1'
        NAME_TRANSFER = 'PRUEBA1'
        NAME_BUCKET_AWS = 'mi-bucket-aws-1'
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
                    // Recupera las credenciales de AWS desde Cloud Secret Manager
                    def awsCredentials = sh(script: "gcloud secrets versions access latest --secret=${NAME_SECRET}", returnStdout: true).trim()
                    // Ruta al archivo donde se guardarán las credenciales
                    def awsCredentialsFilePath = "${env.WORKSPACE}/aws_credentials.json"
                    // Escribir las credenciales en el archivo
                    writeFile file: awsCredentialsFilePath, text: awsCredentials
        
                    // Verificar si ya existe un transfer job con el mismo nombre
                    def existingJob = sh(script: "gcloud transfer jobs describe ${NAME_TRANSFER} --format='value(name)'", returnStdout: true, returnStatus: true)
                    def valor = existingJob == 0 ? 'update' : 'create'
                    
                    sh """
                        gcloud transfer jobs ${valor} ${NAME_TRANSFER} \
                        --source-creds-file=${awsCredentialsFilePath} \
                        --overwrite-when=different \
                        --schedule-repeats-every=1h \
                        --schedule-starts="2024-04-02T14:00:00Z"
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

