pipeline {
    agent any

    environment {
        PROJECT_ID = 'devioz-pe-dev-analitica'
        ENVIRONMENT = 'dev'
        GCP_SERVICE_ACCOUNT = 'devioz-corporativo-gcp-devops-analitica-dev'
        AWS_SERVICE_ACCOUNT = 'AWS_SECRET_ID'
        GCP_LOCATION = 'us-central1'
        NAME_BUCKET_GCP = 'mi-bucket-gcp-1'
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
            // Recupera las credenciales de AWS desde Cloud Secret Manager
            def awsCredentials = sh(script: 'gcloud secrets versions access latest --secret=aws_Cred', returnStdout: true).trim()
            echo "Credenciales de AWS: ${awsCredentials}"
            
            // Ruta al archivo donde se guardarán las credenciales
            def awsCredentialsFilePath = "${env.WORKSPACE}/aws_credentials.json"
            
            // Escribir las credenciales en el archivo
            writeFile file: awsCredentialsFilePath, text: awsCredentials
            echo "Archivo de credenciales de AWS: ${awsCredentialsFilePath}"
            
            // Crea la transferencia de datos utilizando las credenciales recuperadas
         sh """
    gcloud transfer jobs update transferJobs/2347364665222102168\
    --source-creds-file=${awsCredentialsFilePath} \
    --overwrite-when=different \
    --schedule-repeats-every=2h \
    --schedule-repeats-until=2025-12-31
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
