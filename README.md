Readme file explained in a  simple terms so you can follow every step even if you have no technical background.  

---

## **Overview**  
This guide helps you set up and run a basic web app on **Microsoft Azure** (a cloud service). The process involves:  
1. **Setting up prerequisites** (things you need before you start).  
2. **Deploying the infrastructure** (setting up the cloud environment).  
3. **Publishing the web app** (putting the website online).  
4. **Validating the web app** (checking if it's working).  
5. **Optional: Using Service Connector** (a more advanced way to connect to a database).  
6. **Cleaning up** (removing everything once you're done to avoid extra costs).  

---

## **Step 1: Set Up Prerequisites**  
Before starting, you need:  
✅ **An Azure account** – Sign up here: [Azure Free Account](https://azure.microsoft.com/free/)  
✅ **Azure CLI** – A tool that lets you interact with Azure from your computer. Install it here: [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)  
✅ **Bicep tools** – These help deploy the setup automatically. Install them here: [Bicep Tools](https://learn.microsoft.com/azure/azure-resource-manager/bicep/install)  

---

## **Step 2: Deploy the Infrastructure**  
This step creates the environment where your web app will run.

1️⃣ **Open a command-line tool** (like PowerShell or Terminal) on your computer.  
2️⃣ **Navigate to the folder where this project is saved** on your computer.  
3️⃣ **Log in to Azure** and select your account (if needed):  
   ```bash
   az login 
   az account set --subscription 3b80f884xxxxxxxxxx
   ```
   👉 This step connects your computer to your Azure account.  

4️⃣ **Update the configuration file** (`parameters.json`).  
   - Open the file.  
   - Fill in details like a **base name**, **database username**, and **password**.  
   - The password must meet security rules (e.g., uppercase, lowercase, numbers).  

5️⃣ **Run these commands to set up the cloud environment:**  
   ```Powershell
$LOCATION = "ukwest"           # or "UK West" if the region name requires a space (enclose in quotes)
$BASE_NAME = "mikewebapp"       # a short, unique name
$RESOURCE_GROUP = "mikeresourcegp"  # name for your resource group

az group create --location $LOCATION --resource-group $RESOURCE_GROUP

  $RESOURCE_GROUP = "mikeresourcegp"
$BASE_NAME = "mikewebapp"

az deployment group create --template-file "./infra-as-code/bicep/main.bicep" `
     --resource-group $RESOURCE_GROUP `
     --parameters "@./infra-as-code/bicep/parameters.json" `
     --parameters "baseName=$BASE_NAME"

   ```
   👉 This creates all the necessary components in **Azure**, like the database and web hosting environment.

---

## **Step 3: Publish the Web App**  
Now that the environment is ready, let's put the website online.  

1️⃣ Run the following command:  
   ```Powershell 

$LOCATION = "ukwest"                # or "UK West" if the region name requires a space (enclose in quotes)
$BASE_NAME = "mikewebapp"           # a short, unique name
$RESOURCE_GROUP = "mikeresourcegp"  # name for your resource group

az group create --location $LOCATION --resource-group $RESOURCE_GROUP

$APPSERVICE_NAME = "app-$BASE_NAME"

az webapp deploy --resource-group $RESOURCE_GROUP `
                 --name $APPSERVICE_NAME `
                 --type zip `
                 --src-url "https://raw.githubusercontent.com/Azure-Samples/app-service-sample-workload/main/website/SimpleWebApp.zip"


   ```
   👉 This uploads a sample website from GitHub to **Azure App Services**.  

---

## **Step 4: Validate the Web App**  
Now, check if your website is working.  

1️⃣ Run this command:  
   ```Powershell
$APPSERVICE_URL = "https://$APPSERVICE_NAME.azurewebsites.net"
echo $APPSERVICE_URL

   ```  
2️⃣ Open the link in your web browser to see your web app live! 🎉  

---

## **Step 5 (Optional): Connect the Web App to the Database Securely**  
Right now, your web app connects to the database using a **connection string** (like a password). This step improves security using **Service Connector**.

### How to do it:
1️⃣ Open **Azure Cloud Shell** (a terminal inside Azure).  
2️⃣ Run these commands to configure a secure connection:  
   ```PowerShell
   # Enable strict mode and halt on any errors
Set-StrictMode -Version Latest
$ErrorActionPreference = "Stop"

# Define your parameters (replace placeholders with actual values)
$LOCATION      = "westus3"
$BASE_NAME     = "<your-base-name>"         # e.g., "mikewebapp"
$RESOURCE_GROUP = "<your-group-name>"        # e.g., "mikeresourcegp"
$APPSERVICE_NAME = "app-$BASE_NAME"           # e.g., "app-mikewebapp"

# Retrieve resource IDs from previous deployments
$RESOURCEID_DATABASE = az deployment group show `
    -g $RESOURCE_GROUP `
    -n databaseDeploy `
    --query properties.outputs.databaseResourceId.value `
    -o tsv

$RESOURCEID_WEBAPP = az deployment group show `
    -g $RESOURCE_GROUP `
    -n webappDeploy `
    --query properties.outputs.appServiceResourceId.value `
    -o tsv

$USER_IDENTITY_WEBAPP_CLIENTID = az deployment group show `
    -g $RESOURCE_GROUP `
    -n webappDeploy `
    --query properties.outputs.appServiceIdentity.value `
    -o tsv

$USER_IDENTITY_WEBAPP_SUBSCRIPTION = az deployment group show `
    -g $RESOURCE_GROUP `
    -n webappDeploy `
    --query properties.outputs.appServiceIdentitySubscriptionId.value `
    -o tsv

# Output retrieved values for verification
Write-Host "App Service Name: $APPSERVICE_NAME"
Write-Host "Database Resource ID: $RESOURCEID_DATABASE"
Write-Host "WebApp Resource ID: $RESOURCEID_WEBAPP"
Write-Host "WebApp Client Identity: $USER_IDENTITY_WEBAPP_CLIENTID"
Write-Host "WebApp Subscription ID: $USER_IDENTITY_WEBAPP_SUBSCRIPTION"

# (Optional) Delete the old connection string
az webapp config appsettings delete `
    --name $APPSERVICE_NAME `
    --resource-group $RESOURCE_GROUP `
    --setting-names AZURE_SQL_CONNECTIONSTRING

# Install (or upgrade) the Service Connector extension
az extension add --name serviceconnector-passwordless --upgrade

# Set up the secure connection between the WebApp and the Database
az webapp connection create sql `
    --connection sql_adventureconn `
    --source-id $RESOURCEID_WEBAPP `
    --target-id $RESOURCEID_DATABASE `
    --client-type dotnet `
    --user-identity "client-id=$USER_IDENTITY_WEBAPP_CLIENTID subs-id=$USER_IDENTITY_WEBAPP_SUBSCRIPTION"


   ```
   👉 This replaces the old database connection method with a **more secure** approach.

---

## **Step 6: Clean Up (Remove Everything When You're Done)**  
To **avoid being charged** for unused resources, delete everything when you're finished testing.  

Run this command:  
```PowerShell

# Delete the resource group without prompting for confirmation
az group delete --name $RESOURCE_GROUP --yes

```
👉 This removes all the resources you created on Azure.

---

## **Final Summary**  
- You **set up the prerequisites** (Azure account, CLI, Bicep).  
- You **deployed the infrastructure** (created a cloud environment).  
- You **published a web app** (uploaded and launched the site).  
- You **validated the app** (checked if it’s working).  
- (Optional) You **secured the database connection**.  
- You **cleaned up resources** (to avoid extra costs).  

Now you’ve successfully deployed a web app on **Microsoft Azure**! 🚀 Let me know if you need any help. 😊