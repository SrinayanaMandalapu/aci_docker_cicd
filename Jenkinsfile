pipeline {
    agent any

    environment {
        PYTHON = "C:\\Users\\manda\\AppData\\Local\\Programs\\Python\\Python312"

        // Folder where az.cmd lives (change according to `where az` result)
        AZ_CLI = "C:\\Program Files (x86)\\Microsoft SDKs\\Azure\\CLI2\\wbin"

        // Add Azure CLI + Python to PATH
        PATH = "${AZ_CLI};${PYTHON};${PYTHON}\\Scripts;${env.PATH}"

        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        IMAGE_NAME  = 'srinayana20/aci_docker_cicd'

        GIT_REPO = 'https://github.com/SrinayanaMandalapu/aci_docker_cicd'
        BRANCH   = 'main'

        AZURE_RG             = 'my-aci-rg'
        AZURE_LOCATION       = 'eastus'
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
                bat "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                bat """
                    docker login -u ${DOCKERHUB_CREDENTIALS_USR} -p ${DOCKERHUB_CREDENTIALS_PSW}
                    docker push ${IMAGE_NAME}:latest
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
