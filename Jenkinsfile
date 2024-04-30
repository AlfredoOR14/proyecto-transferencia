pipeline {
    agent any

    environment {
        PROJECT_ID = 'devioz-pe-test-analitica'
        NAME_SECRET = 'awss'
        GCP_LOCATION = 'us-central1'
        NAME_BUCKET_GCP = 'mi-bucket-gcp-gcp1'
        GCP_SERVICE_ACCOUNT = 'devioz-pe-test-analitica'
        NAME_TRANSFER = 'TRANSFER2'
        NAME_BUCKET_AWS = 'mi-bucket-aws-1'
    }

    stages {
        stage('Descarga de Fuentes') {
            steps {
                deleteDir()
                checkout scm
            }
        }
/*
        stage('Creación de cuenta de servicio') {
            steps {
                script {
                    echo 'Creando cuenta de servicio en Google Cloud...'
                    sh "gcloud iam service-accounts create ${SERVICE_ACCOUNT_NAME} --description='Service account for data transfer' --display-name='Data Transfer Service Account'"
                    sh "gcloud projects add-iam-policy-binding ${PROJECT_ID} --member=serviceAccount:${SERVICE_ACCOUNT_NAME} --role=roles/storage.admin"
                }
            }
        }
*/
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
                    echo 'Iniciando la etapa de Creacion de Bucket en GCP..'
                    sh "gcloud config set project ${PROJECT_ID}"
                    def bucketExists = sh(script: "gsutil ls gs://${NAME_BUCKET_GCP}", returnStatus: true)
                    if (bucketExists != 0) {
                        sh "gsutil mb -p ${PROJECT_ID} -l ${GCP_LOCATION} gs://${NAME_BUCKET_GCP}"
                        echo 'Bucket en GCP creado correctamente.'
                    } else {
                        echo "El bucket gs://${NAME_BUCKET_GCP} ya existe. No se creará otro."
                    }
                }
            }
        }

        stage('Creacion de trasferencia de datos de AWS a GCP') {
            steps {
                script {
                    def awsCredentials = sh(script: "gcloud secrets versions access latest --secret=${NAME_SECRET}", returnStdout: true).trim()
                    def awsCredentialsFilePath = "${env.WORKSPACE}/aws_credentials.json"
                    writeFile file: awsCredentialsFilePath, text: awsCredentials

                    def command = "gcloud transfer jobs describe ${NAME_TRANSFER} --format='value(name)'"
                    def existingJob = sh(script: command, returnStdout: true, returnStatus: true)

                    command = existingJob == 0 ? "gcloud transfer jobs update ${NAME_TRANSFER}" : "gcloud transfer jobs create s3://${NAME_BUCKET_AWS} gs://${NAME_BUCKET_GCP} --name=${NAME_TRANSFER}"

                    sh """
                    ${command} \
                    --source-creds-file=${awsCredentialsFilePath} \
                    --overwrite-when=different \
                    --schedule-repeats-every=1d \
                    --schedule-starts="2024-04-11T12:30:00Z" \
                    --impersonate-service-account=devioz-pe-test-analitica@devioz-pe-test-analitica.iam.gserviceaccount.com \
                    --schedule-repeats-until="2024-07-31T13:30:00Z" 
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

