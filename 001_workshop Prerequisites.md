# 001_workshop Prerequisites

## Minimal Laptop requirements

A Mac, Windows 10 or Linux Laptop 

-You should be able to connect this laptop to a wireless network.
-Windows users should install the [Windows subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/about)

Install the following on your laptop:

- [Java 8 SDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

- The [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Docker](https://docs.docker.com/install/) 
- Install ssh (if it isn’t there yet by default on your laptop OS)
- A good, multiple window terminal (iTerm-2 on Mac, Powershell or  ConEmu on Windows, Terminator on Linux)
- You should be able to to clone a git repo from GitHub
- No prior knowledge of Azure is required, just an open mind and curiosity about learning Azure
- Basic understanding of Docker
- Basic exposure to terminal / command line



## Azure prerequisites 

>These prerequisites require an azure account - An Azure trial account will be >provided if you don’t have one.  Please check with your instructor.

### Create an SSH public-private key pair

We'll need an SSH key pair to securely t connect to the Linux VMs we'll be working with in Azure.  

Follow the steps in [this tutorial to create a key pair](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-ssh-keys-detailed)  


### Create an Azure service principal to work with VM agents

As part of this tutorial we'll need to create a Service Principal, which is a way to securely communicate between designated Azure resources with roles-based access control as part of an Azure subscription .  

Follow the steps in [this tutorial to create a service principal using the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest) 

**NOTE:** Keep the default service principal role as Contributor, which is sufficient for working with the Azure resources in this workshop.
