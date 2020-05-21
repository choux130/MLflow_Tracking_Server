# MLflow Tracking Server

### Key Words
<code>MLflow</code>, <code>Docker</code>, <code>SSH</code>, <code>Azure Web App</code>, <code>Azure Registry</code>, <code>Auzre Blob Storage</code>

### Reference
* [Project to deploy MLflow Tracking Server on an Azure Web App for Containers (Linux).](https://github.com/marvinbuss/mlflow-on-azure)
* [Containerize your whole Data Science Environment (or anything you want) with Docker-Compose](https://towardsdatascience.com/containerize-your-whole-data-science-environment-or-anything-you-want-with-docker-compose-e962b8ce8ce5)

### Introduction 
This project is about how to host MLflow on Azure as a web app and connect Azure Blob Storage to MLflow for generated model artifacts. 

### Steps
1. Create a "Storage Account" in Azure. The place where model aritacts are going to save. [Create a BlockBlobStorage account](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-create-account-block-blob?tabs=azure-portal)
2. Fill in all the credential in <code>Dockerfile</code> as environment variables.   
3. Test the deployment locally using 
    ```bash
    docker build -t mlflow_tracking_image .
    docker run -p 5000:5000 -p 2222:2222 --name mlflow_tracking_container  mlflow_tracking_image:latest
    ```   
    Open [localhost:5000](localhost:5000) to make sure the MLflow ui show up correctly. And test if the running container can be connected through SSH. If the error message shown up, <code>Unable to negotiate with ::1 port 2222: no matching cipher found. Their offer: aes128-cbc,3des-cbc,aes256-cbc</code>, then take a look at this post. [ssh error: unable to negotiate with IP: no matching cipher found](https://ma.ttias.be/ssh-error-unable-negotiate-ip-no-matching-cipher-found/)
    ```bash
    ssh root@localhost -p 2222
    ```
4. Create a "Azure Registry" in Azure if it has not been created. [Quickstart: Create a private container registry using the Azure portal](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal)
5. Push image to the created Azure Registry. [Push your first image to a private Docker container registry using the Docker CLI](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli)
5. Create "Web App" in Azure using the image in Azure Registry. [Deploying Containerised Apps to Azure Web App for Containers](https://chrissainty.com/containerising-blazor-applications-with-docker-deploying-containerised-apps-to-azure-web-app-for-containers/)

### Quick Test
1. Set environment variables, <code>MLFLOW_TRACKING_URI</code> and <code>AZURE_STORAGE_CONNECTION_STRING</code>. 

    Linux and MacOS
    ```bash
    export MLFLOW_TRACKING_URI=https://<web-app-name>.azurewebsites.net
    export AZURE_STORAGE_CONNECTION_STRING='<connection-string>'
    ```
    Windows
    ```bash
    set MLFLOW_TRACKING_URI=https://<web-app-name>.azurewebsites.net
    set AZURE_STORAGE_CONNECTION_STRING='<connection-string>'
    ```
2. Have python package installed.
   ```bash
   pip install mlflow==1.7.2
   pip install azure-storage-blob==2.1.0
   ```
3. Open Python and make sure the environment variables have been correctly passed. 
    ```python
    import os 
    print(os.getenv('MLFLOW_TRACKING_URI'))
	print(os.getenv('AZURE_STORAGE_CONNECTION_STRING'))
    ```
4. Try to log parameters, metric and artifacts to the tracking server.
    ```python
    import mlflow
    
    mlflow.start_run()
    mlflow.log_param("param1", 500)
    mlflow.log_metric("foo", 100)
 
    with open("output.txt", "w") as f: f.write("Hello world!")
    mlflow.log_artifact("output.txt")   
    mlflow.end_run()
    ```
5. Go to the web app (<code>https://\<web-app-name\>.azurewebsites.net</code>) and see if the run has been logged and go to blob storage to make sure the artifacts have been saved in the container.
    
