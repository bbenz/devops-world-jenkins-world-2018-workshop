# Exercise 003

## Deploying Artifacts with Azure plugins
Now that we've deployed Jenking to Azure VMs and AKS, we'll return to our VM installation to deploy some artifacts using the plugins we installed and configured.  

We'll use the Jenkins [Azure VM Agents plugin](https://plugins.jenkins.io/azure-vm-agents) in our Jenkins VM installation to to add capacity with more Linux virtual machines.

In this exercise, you will:


> * Configure the Agent plugin to create resources in your Azure subscription
> * Set the compute resources available to each agent
> * Set the operating system and tools installed on each agent
> * Create a new Jenkins freestyle job
> * Run the job on an Azure VM agent


## Prerequisites
Please see the prerequisites doc  in this repo for first steps including creating an Azure account and pre-installing required tools and packages.

We have codes for free Azure accounts/credits if you don't have an Azure
subscription - please ask an instructor.

# Part 1 - Jenkins Agents on Azure 

## Steps

### Configure agent resources

Let's configure a template for use to define an Azure VM agent. This template defines the compute resources each agent has when created.  OPen Jenkins on your VM and go to the installed and configured plugin, then: 

1. Select **Add** next to **Add Azure Virtual Machine Template**.
1. Enter `defaulttemplate` for the **Name**
1. Enter `ubuntu` for the **Label**
1. Select the desired [Azure region](https://azure.microsoft.com/regions/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) from the combo box.
1. Select a [VM size](/azure/virtual-machines/linux/sizes) from the drop-down under **Virtual Machine Size**. A general-purpose `Standard_DS1_v2` size is fine for this tutorial.   
1. Leave the **Retention time** at `60`. This setting defines the number of minutes Jenkins can wait before it deallocated idle agents. Specify 0 if you do not want idle agents to be removed automatically.

  
### Configure agent operating system and tools

In the **Image Configuration** section of the plugin configuration, select **Ubuntu 16.04 LTS**. Check the boxes next to **Install Git (Latest)**, **Install Maven (V3.5.0)**, and **Install Docker** to install these tools on newly created agents.


Select **Add** next to **Admin Credentials**, then select **Jenkins**. Enter a username and password used to sign in to the agents, making sure they satisfy the [username and password policy](/azure/virtual-machines/linux/faq#what-are-the-username-requirements-when-creating-a-vm) for administrative accounts on Azure VMs.

Select **Verify Template** to verify the configuration and then select **Save** to save your changes and return to the Jenkins dashboard.

### Create a job in Jenkins

1. Within the Jenkins dashboard, click **New Item**. 
1. Enter `demoproject1` for the name and select **Freestyle project**, then select **OK**.
1. In the **General** tab, choose **Restrict where project can be run** and type `ubuntu` in **Label Expression**. You see a message confirming that the label is served by the cloud configuration created in the previous step. 
  
1. In the **Source Code Management** tab, select **Git** and add the following URL into the **Repository URL** field: `https://github.com/spring-projects/spring-petclinic.git`
1. In the **Build** tab, select **Add build step**, then **Invoke top-level Maven targets**. Enter `package` in the **Goals** field.
1. Select **Save** to save the job definition.

### Build the new job on an Azure VM agent

1. Go back to the Jenkins dashboard.
1. Select the job you created in the previous step, then click **Build now**. A new build is queued, but does not start until an agent VM is created in your Azure subscription.
1. Once the build is complete, go to **Console output**. You see that the build was performed remotely on an Azure agent.



# Part 2 - CI and CD to Azure App Service

Next, we'll set up continuous integration and deployment (CI/CD) of a sample Java web app developed with the [Spring Boot](http://projects.spring.io/spring-boot/) framework to [Azure App Service Web App on Linux](/azure/app-service/containers/app-service-linux-intro) using Jenkins.

## Steps:

> 
> * Define a Jenkins job to build Docker images from a GitHub repo when a new commit is pushed.
> * Define a new Azure Web App for Linux and configure it to deploy Docker images pushed to Azure Container registry.
> * Configure the Azure App Service Jenkins plug-in.
> * Deploy the sample app to Azure App Service with a manual build.
> * Trigger a Jenkins build and update the web app by pushing changes to GitHub.

### Configure GitHub and Jenkins

Set up Jenkins to receive [GitHub webhooks](https://developer.github.com/webhooks/) when new commits are pushed to a repo in your account.

1. Select **Manage Jenkins**, then **Configure System**. In the **GitHub** section, make sure **Manage hooks** is selected and then select **Manage additional GitHub actions** and choose **Convert login and password to token**.
2. Select the **From login and password** radio button and enter your GitHub username and password. Select **Create token credentials** to create a new [GitHub Personal Access Token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/).   
  
3.  Select the newly created token from the **Credentials** drop down in the GitHub Servers configuration. Select **Test connection** to verify that the authentication is working.   
  
> [!NOTE]
> If your GitHub account has two-factor authentication enabled,  create the token on GitHub and configure Jenkins to use it. Review the [Jenkins GitHub plug-in](https://wiki.jenkins.io/display/JENKINS/Github+Plugin) documentation for full details.

### Fork the sample repo and create a Jenkins job 

1. Open the [Spring Boot sample application repo](https://github.com/spring-guides/gs-spring-boot-docker) and fork it to your own GitHub account by selecting **Fork** in the top right-hand corner.   
  
1. In the Jenkins web console, select **New Item**, give it a name **MyJavaApp**, select **Freestyle project**, then select **OK**.   
    
2. Under the **General** section, select **GitHub** project and enter your forked repo URL such as https://github.com/raisa/gs-spring-boot-docker
3. Under the **Source code management**  section, select **Git**, enter your forked repo `.git` URL such as https://github.com/raisa/gs-spring-boot-docker.git
4. Under the **Build Triggers** section, select **GitHub hook trigger for GITscm polling**.
5. Under the **Build** section, select **Add build step** and choose **Invoke top-level Maven targets**. Enter `package` in the **Goals** field.
6. Select **Save**. You can test your job by selecting **Build Now** from the project page.

### Configure Azure App Service 

1. Using the Azure CLI or [Cloud Shell](/azure/cloud-shell/overview), create a new [Web App on Linux](/azure/app-service/containers/app-service-linux-intro). The web app name in this tutorial is `myJavaApp`, but you need to use a unique name for your own app.
   
    ```azurecli-interactive
    az group create --name myResourceGroupJenkins --location westus
    az appservice plan create --is-linux --name myLinuxAppServicePlan --resource-group myResourceGroupJenkins 
    az webapp create --name myJavaApp --resource-group myResourceGroupJenkins --plan myLinuxAppServicePlan --runtime "java|1.8|Tomcat|8.5"
    ```

2. Create an [Azure Container Registry](/azure/container-registry/container-registry-intro) to store the Docker images built by Jenkins. The container registry name used in this tutorial is `jenkinsregistry`, but you need to use a unique name for your own container registry. 

    ```azurecli-interactive
    az acr create --name jenkinsregistry --resource-group myResourceGroupJenkins --sku Basic --admin-enabled
    ```
3. Configure the web app to run Docker images pushed to the container registry and specify that the app running in the container listens for requests on port 8080.   

    ```azurecli-interactive
    az webapp config container set -c jenkinsregistry/webapp --resource-group myResourceGroupJenkins --name myJavaApp
    az webapp config appsettings set --resource-group myResourceGroupJenkins --name myJavaApp --settings PORT=8080
    ```

### Configure the Azure App Service Jenkins plug-in

1. In the Jenkins web console, select the **MyJavaApp** job you created and then select **Configure** on the left hand of the page.
2. Scroll down to **Post-build Actions**, select **Add post-build action**, and choose **Publish an Azure Web App**.
3. Under **Azure Profile Configuration**, select **Add** next to **Azure Credentials** and choose **Jenkins**.
4. In the **Add Credentials** dialog, select **Microsoft Azure Service Principal** from the **Kind** drop-down.
5. Create an Active Directory Service principal from the Azure CLI.
    
    ```azurecli-interactive
    az ad sp create-for-rbac --name jenkins_sp --password secure_password
    ```

    ```json
    {
        "appId": "BBBBBBBB-BBBB-BBBB-BBBB-BBBBBBBBBBB",
        "displayName": "jenkins_sp",
        "name": "http://jenkins_sp",
        "password": "secure_password",
        "tenant": "CCCCCCCC-CCCC-CCCC-CCCCCCCCCCC"
    }
    ```
6. Enter the credentials from the service principal into the **Add credentials** dialog. If you don't know your Azure subscription ID, you can query it from the CLI:
     
     ```azurecli-interactive
     az account list
     ```

     ```json
        {
            "cloudName": "AzureCloud",
            "id": "AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA",
            "isDefault": true,
            "name": "Visual Studio Enterprise",
            "state": "Enabled",
            "tenantId": "CCCCCCCC-CCCC-CCCC-CCCC-CCCCCCCCCCC",
            "user": {
            "name": "raisa@fabrikam.com",
            "type": "user"
            }
     ```

   
6. Verify the service principal authenticates with Azure by selecting **Verify Service Principal**. 
7. Select **Add** to save the credentials.
8. Select the service principal credential you just added from the **Azure Credentials** drop-down when you are back to the **Publish an Azure Web App** configuration.
9. In **App Configuration**, choose your resource group and web app name from the drop-down.
10. Select the **Publish via Docker** radio button.
11. Enter `complete/Dockerfile` for **Dockerfile path**.
12. Enter `https://jenkinsregistry.azurecr.io` in the **Docker registry URL** field.
13. Select **Add** next to **Registry Credentials**. 
14. Enter the admin username for the Azure Container Registry you created for the **Username**.
15. Enter the password for the Azure Container registry in the **Password** field. You can get your username and password from the Azure portal or through the following CLI command:

    ```azurecli-interactive
    az acr credential show -n jenkinsregistry
    ```
    
15. Select **Add** to save the credential.
16. Select the newly created credential from the **Registry credentials** drop-down in the **App Configuration** panel for the **Publish an Azure Web App**. The finished post-build action should look like the following image:   
   
17. Select **Save** to save the job configuration.

### Deploy the app from GitHub

1. From the Jenkins project, select **Build Now** to deploy the sample app to Azure.
2. Once the build completes, your app is live on Azure at its publishing URL, for example http://myjavaapp.azurewebsites.net.   
  
### Push changes and redeploy

1. From your Github fork, browse on the web to  `complete/src/main/java/Hello/Application.java`. Select the **Edit this file** link from the right-hand side of the GitHub interface.
2. Make the following change to the `home()` method and commit the change to the repo's master branch.
   
    ```java
    return "Hello Docker World on Azure";
    ```
3. A new build starts in Jenkins, triggered by the new commit on the `master` branch of the repo. Once it completes, reload your app on Azure.     
     

