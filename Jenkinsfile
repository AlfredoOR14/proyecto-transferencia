pipeline {
    agent any

    environment {
        PROJECT_ID = 'devioz-pe-dev-analitica'
        NAME_SECRET = 'awss'
        GCP_SERVICE_ACCOUNT = 'devioz-corporativo-gcp-devops-analitica-dev'
        GCP_LOCATION = 'us-central1'
        NAME_BUCKET_GCP = 'mi-bucket-gcp-gcp-gcp'
        NAME_BUCKET_S3 = 'mi-bucket-aws-1'
        NAME_TRANSFER = 'PRUEBAS11'
        NAME_BUCKET_AWS = 'mi-bucket-aws-12'
    }

      stages {
            stage('Descarga de Fuentes') {
                steps {
                    deleteDir()
                    checkout scm
                }
            }

            stage('Activando Service Account') {
                steps {
                    echo 'Iniciando la etapa de Activando Service Account...'
                    withCredentials([file(credentialsId: "${GCP_SERVICE_ACCOUNT}", variable: 'SECRET_FILE')]) {
                        sh "gcloud auth activate-service-account --key-file=\$SECRET_FILE"
                    }
                    echo 'Service Account activada correctamente.'
                }
            }

            stage('Create Bucket in GCP') {
                steps {
                    script {
                        echo 'Iniciando la etapa de Creacion de Bucket en GCP...'
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
                    
                         command = existingJob == 0 ? "gcloud transfer jobs update ${NAME_TRANSFER}" : "gcloud transfer jobs create s3://${NAME_BUCKET_S3} gs://${NAME_BUCKET_GCP} --name=${NAME_TRANSFER}"
                    
                    sh """
                        ${command} \
                        --source-creds-file=${awsCredentialsFilePath} \
                        --overwrite-when=different \
                        --schedule-repeats-every=1h \
                        --schedule-starts="2024-04-11T19:30:00Z" \
                        --schedule-end-date="2024-04-30" \
                        --schedule-end-time="12:30:00"
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
