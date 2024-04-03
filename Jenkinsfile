pipeline {
    agent any

    environment {
        PROJECT_ID = 'devioz-pe-dev-analitica'
        NAME_SECRET = 'aws_Cred'
        GCP_SERVICE_ACCOUNT = 'devioz-corporativo-gcp-devops-analitica-dev'
        GCP_LOCATION = 'us-central1'
        NAME_BUCKET_GCP = 'mi-bucket-gcp-2'
        NAME_BUCKET_S3 = 'mi-bucket-aws-1'
        AWS_REGION = 'us-east-1'
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
               
           stage('Transferencia de datos de AWS S3 a GCP') {
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
            
                        // Copiar los datos de AWS S3 a una carpeta local temporal
                        sh """
                            aws s3 cp s3://${NAME_BUCKET_S3} ${env.WORKSPACE}/temp --recursive --quiet --region=${AWS_REGION}
                        """
            
                        // Verificar si la transferencia fue exitosa
                        def transferCheck = sh(script: "ls ${env.WORKSPACE}/temp", returnStatus: true)
                        if (transferCheck == 0) {
                            echo "Datos de AWS S3 transferidos con éxito a la carpeta temporal."
            
                            // Copiar los datos de la carpeta temporal a GCS
                            sh """
                                gsutil -m cp -r ${env.WORKSPACE}/temp gs://${NAME_BUCKET_GCP}
                            """
            
                            // Verificar si la transferencia a GCS fue exitosa
                            def gcsCheck = sh(script: "gsutil ls gs://${NAME_BUCKET_GCP}", returnStatus: true)
                            if (gcsCheck == 0) {
                                echo "Datos transferidos con éxito de AWS S3 a GCS."
                            } else {
                                error "La transferencia de datos a GCS ha fallado."
                            }
                        } else {
                            error "La transferencia de datos desde AWS S3 ha fallado."
                        }
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
