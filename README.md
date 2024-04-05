# Pre-requisites
1. You have Python installed
2. You have installed prompt flow using pip  
2. You have created Azure OpenAI Service and deployed relevant model(s)
3. You have Docker installed
4. You have created Azure Container Registry  
5. You have Created Azure App Service  
6. You have Azure CLI installed  
7. You have an Entra ID user or service principal with relevant RBAC roles  


# Preparation  
1. Create a .env file based on .env.example. Update the values for the environment variables.  
2. Create a connection.yaml file based on connection-sample.yaml. Update the values for connection variables.  
3. Run command "pip install -r requirements.txt"  
4. Run command "pf connection create -f connection.yaml"  

# Flow testing
Run command "pf flow test --flow .  # ". for current directory"   

# Prepare Docker image and upload to Azure Container Registry
1. Run command "pf flow build --source . --output <your-output-dir> --format docker"  
2. Go to <your-output-dir>  
3. Copy connection.yaml from your source code to connections folder  
4. Inside <your-output-dir>, run command "docker build ."
5. Run command "az login" either with your Entra ID user or with service principal  
6. Run command "az acr login --name <your Azure Container registry name>"
7. Enable admin account for your Azure Container Registry "az acr update -n <your Azure Container registry name> --admin-enabled true"  
8. Run command "docker login <your Azure Container registry name>.azurecr.io". Use your service principal client ID and secret as user and password for login  
9. Run command "docker build . -t <your Azure Container registry name>.azureacr.io/<Name of your flow>"  
10. Run command "docker push <your Azure Container registry name>.azurecr.io/<Name of your flow>"
11. Start docker container for testing with the command "docker run -p 8080:8080 -e AZURE_OPENAI_API_KEY=<your AOAI API key> -e AZURE_OPENAI_API_BASE=https://<your AOAI deployment name>.openai.azure.com/ -e AZURE_OPENAI_API_TYPE=azure  <your Azure Container registry name>.azurecr.io/<Name of your flow>"
12. Test docker container with command "curl http://localhost:8080/score --data '{"text":"hello world"}' -X POST  -H "Content-Type: application/json""

# Deploy docker image to Azure App Service Web App
1. Login to Azure portal and create a resource of type Web App  
2. Give Web App a name
3. In "Publish" field, select "Container"  
4. Select "Linux" and select Azure region for subsequent fields  
5. Select pricing plan and other values  
6. Change the tab to "Conatiner", select "Image Source" as "Azure Container Service", select "Single Container"  
7. Select your Azure Container Registry, Image and Tag  
8. Create / Deploy the web app  
9. Test prompt flow API using this command -> curl https://<your web app name>.azurewebsites.net/score --data '{"text":"hello world"}' -X POST  -H "Content-Type: application/json"