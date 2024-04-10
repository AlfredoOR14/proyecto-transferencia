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
                script {
                    deleteDir()
                    checkout scm
                }
            }
        }

        stage('Activando Service Account') {
            steps {
                script {
                    echo 'Iniciando la etapa de Activando Service Account...'
                    withCredentials([file(credentialsId: "${GCP_SERVICE_ACCOUNT}", variable: 'SECRET_FILE')]) {
                        sh """\$(gcloud auth activate-service-account --key-file=\$SECRET_FILE)"""
                    }
                    echo 'Service Account activada correctamente.'
                }
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
                        // Recupera las credenciales de AWS desde Cloud Secret Manager
                        def awsCredentials = sh(script: "gcloud secrets versions access latest --secret=${NAME_SECRET}", returnStdout: true).trim()
                        
                        // Ruta al archivo donde se guardarán las credenciales
                        def awsCredentialsFilePath = "${env.WORKSPACE}/aws_credentials.json"
                        // Escribir las credenciales en el archivo
                        writeFile file: awsCredentialsFilePath, text: awsCredentials
                        
                        // Definir la variable COMANDO fuera del bloque if-else
                        def COMANDO
                        def NOMBRE
                        
                        // Verificar si ya existe un transfer job con el mismo nombre
                        def existingJob = sh(script: "gcloud transfer jobs describe ${NAME_TRANSFER} --format='value(name)'", returnStdout: true, returnStatus: true)
                        if(existingJob == 0) {
                            // Si el trabajo ya existe, actualízalo
                            COMANDO = "gcloud transfer jobs update ${NAME_TRANSFER}"
                             NOMBRE = ""
                        } else { 
                            // Si el trabajo no existe, créalo
                            COMANDO = "gcloud transfer jobs create s3://${NAME_BUCKET_S3} gs://${NAME_BUCKET_GCP}"
                            NOMBRE = "--name=${NAME_TRANSFER}"
                        }
                        
                        // Crear o actualizar el trabajo de transferencia
                        sh """
                            ${COMANDO} \
                             ${NOMBRE} \
                            --source-creds-file=${awsCredentialsFilePath} \
                            --overwrite-when=different \
                            --schedule-repeats-every=2h \
                            --schedule-starts="2024-04-03T20:17:00Z" 
                        """
                    }
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

