pipeline {
    agent any

    environment {
        PROJECT_ID = 'devioz-pe-dev-analitica'
        NAME_SECRET = 'awss'
        GCP_SERVICE_ACCOUNT = 'devioz-corporativo-gcp-devops-analitica-dev'
        GCP_LOCATION = 'us-central1'
        NAME_BUCKET_GCP = 'mi-bucket-gcp-gcp'
        NAME_BUCKET_S3 = 'mi-bucket-aws-1'
        NAME_TRANSFER = 'PRUEBAS10'
        NAME_BUCKET_AWS = 'mi-bucket-aws-1'
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

        stage('Creacion de Bucket en GCP') {
            steps {
                echo 'Iniciando la etapa de Creacion de Bucket en GCP...'
                sh "gcloud config set project ${PROJECT_ID}"
                sh "gsutil mb -p ${PROJECT_ID} -l ${GCP_LOCATION} gs://${NAME_BUCKET_GCP}" 
            }
        }

        stage('Creacion de trasferencia de datos de AWS a GCP') {
            steps {
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
                    --schedule-repeats-every=2h \
                    --schedule-starts="2024-04-03T20:17:00Z"
                """
            }
        }

        stage('Limpiando Workspace') {
            steps {
                echo 'Iniciando la etapa de Limpiando Workspace...'
                deleteDir()
                echo 'Workspace limpiado.'
            }
        }
    }
}
