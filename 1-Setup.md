# Setup

In this part you are going to:

1. Create your Microsoft Azure account
2. Connect to the lab environment using SSH
3. Connect to your Microsoft Azure account
4. Deploy your Kubernetes cluster using Azure Container Service
5. Get and save credentials
6. Create an Azure Container Registry

## 1. Create your Microsoft Azure account

*If you already have a Microsoft Azure account that you can use for the hands-on-lab, you can skip this step.*

Open a new **In Private** or **Incognito** browser session and go to <https://www.microsoftazurepass.com>

![](images/azurepassorg/1.png)

![](images/azurepassorg/2.png)

![](images/azurepassorg/3.png)

![](images/azurepassorg/4.png)

![](images/azurepassorg/5.png)

![](images/azurepassorg/6.png)

![](images/azurepassorg/7.png)

![](images/azurepassorg/8.png)

Close the tab on the right (it would lead you to free trial that asks for a Credit Card), 
and use the first tab to continue on with your newly created organizational account.

![](images/azurepassorg/9.png)

![](images/azurepassorg/10.png)

![](images/azurepassorg/11.png)

![](images/azurepassorg/12.png)

![](images/azurepassorg/13.png)

![](images/azurepassorg/14.png)

![](images/azurepassorg/15.png)

![](images/azurepassorg/16.png)

![](images/azurepassorg/17.png)

![](images/azurepassorg/18.png)

![](images/azurepassorg/19.png)

![](images/azurepassorg/20.png)

Your Microsoft Azure account is now ready.

## 2. Connect to the lab environment using SSH

You have been assigned to a SSH endpoint that will allow you to enter the lab environment. The two important information are:

- The port you have been assigned
- The SSH endpoint

The following steps will guide you to create the ssh connection depending on you are running Windows, macOS or Linux on your laptop.

### Windows

1. Download and install putty from [here](https://the.earth.li/~sgtatham/putty/latest/w32/putty-0.68-installer.msi)
2. Open Putty and enter your connection information. Enter a name for the session, click **Save** then **Open**

![Open Putty SSH Session](images/putty.png)


3. Click Yes on the security alert popup that opens
4. Enter root's password: `P@ssw0rd!`
5. You are now connected:

![Putty SSH Session opened](images/putty-ssh-session-opened.png)


### macOS or Linux

Open a terminal and type:

```bash
ssh -p 22XX root@ASK_DNS_TO_PROCTOR.COM
```

When asked, enter the root's password: `P@ssw0rd!`. And you are connected!

## 3. Connect to your Microsoft Azure account

Once connected into the lab environment, you need to authenticate with your Microsoft Azure account using Azure CLI 2.0:

```bash
az login
```

A code will be generated for you. Go to [https://aka.ms/devicelogin](https://aka.ms/devicelogin), enter the code and authenticate with you credentials (the one you have used to create your Microsoft Azure account during step 1.)

![Device Login](images/device-login.png)

![Device Login success](images/device-login-success.png)

## 4. Deploy your Kubernetes cluster using Azure Container Service

Deploying a Kubernetes cluster in Microsoft Azure is really easy using Azure CLI 2.0.

### Set the default Azure account to use

First, list your Azure subscriptions using:

```bash
az account list -o table
```

Copy the *SubscriptionId* of the Azure subscription you want to use and make sure it is the default option using:

```bash
az account set --subscription "REPLACE_WITH_YOUR_SUBSCRIPTION_ID"
```

### Deploy Kubernetes

The first step consists into create a resource group which is a logical entity in Azure that allows to regroup resources that are linked together.

Create a resource group named *devoxx-k8s-rg* in the West Europe Azure datacenter, use:

```bash
az group create --name="devoxx-k8s-rg" --location="westeurope"
```

Open the script `/root/scripts/create-k8s-cluster.sh` with nano:

```bash
nano /root/scripts/create-k8s-cluster.sh
```

And update the option **YOUR_DNS_PREFIX**, for example with YOURNAME-k8s:

```bash
az acs create --resource-group "devoxx-k8s-rg" \
  --location "westeurope" \
  --name "devoxx-k8s" \
  --orchestrator-type "kubernetes" \
  --dns-prefix "YOUR_DNS_PREFIX" \
  --agent-count 2 \
  --master-count 1 \
  --generate-ssh-keys
```

Exit and save the file (CTRL+X then Y then ENTER). 
This script uses Azure CLI 2.0 to generate all the stuff needed for the deployment (SSH keys, Azure Service Principal...) and then deploy the cluster.

Execute the script:

```bash
cd /root/scripts
./create-k8s-cluster.sh
```

The wait for the deployment to be completed.
It can take several minutes.

Once completed, you can browse the [Microsoft Azure Portal](https://portal.azure.com), login with your credentials, browse the resource group where you have deployed Kubernetes and see all the resources that have been created for you:

![Kubernetes in the Microsoft Azure Portal](images/portal-deployment-completed.png)


### Get credentials to connect to the Kubernetes cluster

Once the cluster has been provisionned, you can get the credentials and kubectl config using the following command:

```bash
az acs kubernetes get-credentials -n "devoxx-k8s" -g "devoxx-k8s-rg"
```

It will download everything for you. Then you can test the connection to the cluster using the `kubectl` command:

```bash
root@32322bbcf593:~/scripts# kubectl version
Client Version: version.Info{Major:"1", Minor:"4", GitVersion:"v1.4.5", GitCommit:"5a0a696437ad35c133c0c8493f7e9d22b0f9b81b", GitTreeState:"clean", BuildDate:"2016-10-29T01:38:40Z", GoVersion:"go1.6.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.3", GitCommit:"029c3a408176b55c30846f0faedf56aae5992e9b", GitTreeState:"clean", BuildDate:"2017-02-15T06:34:56Z", GoVersion:"go1.7.4", Compiler:"gc", Platform:"linux/amd64"}
root@32322bbcf593:~/scripts#
root@32322bbcf593:~/scripts#
root@32322bbcf593:~/scripts# kubectl get nodes
NAME                    STATUS                     AGE
k8s-agent-8236386e-0    Ready                      1m
k8s-agent-8236386e-1    Ready                      1m
k8s-master-8236386e-0   Ready,SchedulingDisabled   1m
root@32322bbcf593:~/scripts#
```

If all work well, you should see the client and server version as the output of the `kubectl version` command and the list of nodes as the output of the `kubectl get nodes` command.

**Note: if you have changed the name of the resource group and the name of the cluster in the deployment script, you can list your cluster using:**

```bash
az acs list -o table
```

Your Kubernetes cluster is now up and running on Microsoft Azure!

## 5. Get and save credentials

During the deployment step, the Azure CLI has generated several credentials that you will need in the next parts to automate the delivery pipeline of the application into the Kubernetes cluster.

First, you need to get your Tenant Id. You can get it by listing your Microsoft Azure accounts:

```bash
az account list -o table
```

And the displaying the detailled information of the account you are currently using:

```bash
az account show --subscription SUBSCRIPTION_ID -o table
```

![Get tenant id](images/get-tenant-id.png)


Save the Tenant Id value for later.

Then, you need to get information about the service principal that has been generated during the deployment. 

*Note: a service principal is a kind of service account in Azure Active Directory that allows to authenticate to your subscription in a script, for example.*

You can open the file `/root/.azure/acsServicePrincipal.json` with `nano` to get the identifier of the one that has been generated:

```bash
nano /root/.azure/acsServicePrincipal.json
```

Copy the value of the `client_secret` field and save it for later.

You can also copy the value of the `service_principal` field and use it to display details about this service principal, using:

```bash
az ad sp show --id SERVICE_PRINCIPAL_ID
```

Copy the service principal name (starting by http) and save it for later:

![Get tenant id](images/service-principal-name.png)


Finally, save the `id_rsa` and `id_rsa.pub` SSH private and public keys that have been generated into the `/root/.ssh` folder:

![Get tenant id](images/ssh-keys.png)


## 6. Create an Azure Container Registry

Azure Container Registry is an implementation "as a service" of the open source Docker registry. 

To create a new one, you can also use Azure Container CLI 2.0, with the following command:

```bash
az acr create -n "YOURNAME" -l "westeurope" -g "devoxx-k8s-rg" --admin-enabled true
```

Where:
- `n` is the name of the registry you want to create
- `l` is the datacenter where you want the registry to be created
- `g` is the name of the resource group where you have deployed Kubernetes before
- `--admin-enabled` will allow you to get an admin username / password to login into the registry

Please wait while the registry is deployed...

Then you can get the URL and the username where the registry is available on Azure:

![ACR Creation output](images/acr-create-output.png)


Now you can get the credentials to connect the registry using:

```bash
az acr credential show --name YOURNAME
```

![ACR Creation output](images/acr-credentials.png)


Save the registry URL, login and password for later.

*Note: if you prefer, you can also browse you resource group using the [Microsoft Azure Portal](https://portal.azure.com) and get credentials from the **Access keys** section*:

![ACR Creation output](images/acr-azure-portal.png)

The infrastructure is now running ! You can [go to Part 2](https://github.com/jpbriend/mobile-deposit-api-devoxx2017/blob/master/README.md) and start deploying the application !