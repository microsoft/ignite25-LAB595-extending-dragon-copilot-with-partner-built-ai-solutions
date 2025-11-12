## Deploying to Azure.

1. Build the Docker Image
From the repository root directory, build the Docker image:

```
docker build -f samples/DragonCopilot/Workflow/SampleExtension.Web/Dockerfile -t dragon-extension:latest .
```
> Note: The Dockerfile must be built from the repository root because it references files from both src/Dragon.Copilot.Models/ and samples/DragonCopilot/Workflow/SampleExtension.Web/.

2. Test the docker image Locally (Optional but Recommended)

    ```
    docker run -p 5181:8080 dragon-extension:latest
    ```

3. Execute a test call to the health endpoint of the running docker image in the terminal:
    ```
    curl http://localhost:5181/health
    ```

3. Perform an `az login`

5. Create the needed azure resources using the following powershell script. 
```powershell
$applicationId = Read-Host -Prompt "Please enter your Application (client) ID"
$ExtensionName = "my-extension-@lab.LabInstance.Id"
$ResourceGroup = "@lab.CloudResourceGroup(ResourceGroup1).Name"
$Region = "@lab.CloudResourceGroup(ResourceGroup1).Location"
$EnvironmentSuffix = "dev"
$environmentName = "$ExtensionName-$EnvironmentSuffix-env"
$containerAppName = "$ExtensionName-$EnvironmentSuffix"

az login
az provider register -n Microsoft.App --wait


# Create a resource group
az group create --name $ResourceGroup --location $Region

az provider register -n Microsoft.OperationalInsights --wait

$workspaceName = ($ExtensionName -replace '-','') + "workspace" + $EnvironmentSuffix
az monitor log-analytics workspace create -n $workspaceName --resource-group  $ResourceGroup
$workspaceId = az monitor log-analytics workspace show  -n $workspaceName --resource-group  $ResourceGroup --query customerId -o tsv
$primarySharedKey = az monitor log-analytics workspace get-shared-keys  -n $workspaceName --resource-group  $ResourceGroup --query primarySharedKey -o tsv

# Create a container app environment
az containerapp env create `
            --name $environmentName `
            --resource-group $ResourceGroup `
            --location $Region `
            --logs-workspace-id $workspaceId `
    		--logs-workspace-key $primarySharedKey

# Create a container registry
$registryName = ($ExtensionName -replace '-','') + "acr" + $EnvironmentSuffix
$registryServer = "$registryName.azurecr.io"

az acr create `
    --name $registryName `
    --resource-group $ResourceGroup `
	--location $Region `
	--sku Basic `
	--admin-enabled true

# Get ACR admin credentials
$acrUsername = az acr credential show --name $registryName --resource-group $ResourceGroup --query "username" -o tsv
$acrPassword = az acr credential show --name $registryName --resource-group $ResourceGroup --query "passwords[0].value" -o tsv 

# Tag the built docker image
$imageName = "$registryServer/dragon-extension"
$imageTag = "latest"
$fullImageName = "${imageName}:${imageTag}"

docker tag dragon-extension:latest $fullImageName

# Login to ACR
az acr login --name $registryName

# Push the image
docker push $fullImageName

# Update the TenantId and ClientId to reflect your registered application
az containerapp create `
	--name $containerAppName `
	--resource-group $ResourceGroup `
    --environment $environmentName `
    --image $fullImageName `
	--registry-server $registryServer `
	--registry-username $acrUsername `
    --registry-password $acrPassword `
    --target-port 8080 `
    --ingress external `
    --env-vars "ASPNETCORE_ENVIRONMENT=Production" "ASPNETCORE_URLS=http://+:8080" "Authentication__Enabled=true" `
    	Authentication__TenantId=@lab.CloudSubscription.TenantId `
    	Authentication__ClientId=$applicationId `
        LicenseKey__Enabled=false `
    --cpu 0.25 `
    --memory 0.5Gi `
    --min-replicas 0 `
    --max-replicas 3

# Get the URL
$containerAppUrl = az containerapp show `
	--name $containerAppName `
    --resource-group $ResourceGroup `
    --query "properties.configuration.ingress.fqdn" `
    -o tsv

Write-Host("Your extension is available at: $containerAppUrl/v1/process")
```

4. Using the obtained URL, update your application registration to add a secondary identifier URL

6. Update your manifest to point to the newly deployed URL

8. You can now test your extension using a hosted version of it.