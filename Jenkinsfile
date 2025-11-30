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

            // 1) Login
            bat """
            az login --service-principal --username %AZ_CLIENT_ID% --password %AZ_CLIENT_SECRET% --tenant %AZ_TENANT_ID%
            """

            // 2) Select subscription
            bat """
            az account set --subscription %AZ_SUBSCRIPTION_ID%
            """

            // 3) Try delete old container (if it exists). Ignore error if not found.
            bat """
            az container delete --resource-group my-aci-rg --name flask-aci-app --yes --only-show-errors || echo No existing container to delete
            """

            // 4) Create new container from latest image
            bat """
            az container create ^
              --resource-group my-aci-rg ^
              --name flask-aci-app ^
              --image srinayana20/aci_docker_cicd:latest ^
              --cpu 1 ^
              --memory 1 ^
              --ports 5000 ^
              --dns-name-label flask-aci-app-%BUILD_NUMBER% ^
              --location eastus ^
              --os-type Linux
            """

            // 5) Logout (optional, but clean)
            bat "az logout"
        }
    }
}


    }
}
