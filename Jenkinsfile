pipeline {
    agent any

    environment {
        PYTHON = "C:\\Users\\manda\\AppData\\Local\\Programs\\Python\\Python312"
        PATH = "${PYTHON};${PYTHON}\\Scripts;${env.PATH}"

        // Docker Hub
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        IMAGE_NAME  = 'srinayana20/aci_docker_cicd'

        // Git
        GIT_REPO = 'https://github.com/SrinayanaMandalapu/aci_docker_cicd'
        BRANCH   = 'main'

        // Azure (non-secret)
        AZURE_RG          = 'my-aci-rg'
        AZURE_LOCATION    = 'eastus'
        AZURE_CONTAINER_NAME = 'flask-aci-app'
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Install Dependencies (Build)') {
            steps {
                bat "pip install -r requirements.txt"
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Login & Push to Docker Hub') {
            steps {
                bat """
                    docker login -u ${DOCKERHUB_CREDENTIALS_USR} -p ${DOCKERHUB_CREDENTIALS_PSW}
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker logout
                """
            }
        }

        stage('Deploy to Azure Container Instances') {
            steps {
                withCredentials([
                    string(credentialsId: 'azure-client-id',       variable: 'AZ_CLIENT_ID'),
                    string(credentialsId: 'azure-client-secret',   variable: 'AZ_CLIENT_SECRET'),
                    string(credentialsId: 'azure-tenant-id',       variable: 'AZ_TENANT_ID'),
                    string(credentialsId: 'azure-subscription-id', variable: 'AZ_SUBSCRIPTION_ID')
                ]) {
                    bat """
                    az login --service-principal ^
                        --username %AZ_CLIENT_ID% ^
                        --password %AZ_CLIENT_SECRET% ^
                        --tenant %AZ_TENANT_ID%

                    az account set --subscription %AZ_SUBSCRIPTION_ID%

                    REM Delete existing container if present
                    az container delete ^
                        --resource-group %AZURE_RG% ^
                        --name %AZURE_CONTAINER_NAME% ^
                        --yes ^
                        --only-show-errors

                    REM Create new container
                    az container create ^
                        --resource-group %AZURE_RG% ^
                        --name %AZURE_CONTAINER_NAME% ^
                        --image ${IMAGE_NAME}:${BUILD_NUMBER} ^
                        --cpu 1 ^
                        --memory 1 ^
                        --ports 5000 ^
                        --dns-name-label %AZURE_CONTAINER_NAME%-dns ^
                        --location %AZURE_LOCATION%

                    az logout
                    """
                }
            }
        }
    }
}
