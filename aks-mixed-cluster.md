## Azure Kubernetes Services with Both Linux and Windows Worker nodes

### Pre-requisites
* Experience with Containers & Kubernetes
* An Azure subscription (with Owner role)
* vscode [download](https://code.visualstudio.com/Download)

### The Procedure

1. Install Azure CLI 2.x using [this](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) procedure.
2. Install aks preview extensions

    ```console
    $ az extension add --name aks-preview
    ```

    If already installed, just update it
    ```console
    $ az extension update --name aks-preview
    ```

3.  Register windows preview feature.

    ```console
    $ az feature register --name WindowsPreview --namespace Microsoft.ContainerService
    ```

4.  Wait for 2 minutes and then refresh the registration of Container services (Now to add windows preview features).

    ```console
    $ az provider register --namespace Microsoft.ContainerService
    ```

5.  Create a new resource group for your AKS cluster.

    ```console
    $ az group create --name mixed-aks-group --location southeastasia
    ```

6.  Create the cluster now using AZ AKS command.

    ```console
    $ az aks create \                              
        --resource-group mixed-aks-group \
        --name aks1020 \     
        --node-count 2 \
        --enable-addons monitoring \
        --kubernetes-version 1.14.1 \
        --generate-ssh-keys \
        --windows-admin-password $PASSWORD_WIN \
        --windows-admin-username mahendra \
        --enable-vmss \
        --network-plugin azure
    ```

7.  Once the cluster is created, add windows node pool to cluster.

    ```console
    $ az aks nodepool add \
        --resource-group mixed-aks-group \
        --cluster-name aks1020 \
        --os-type Windows \
        --name winno \
        --node-count 2 \
        --kubernetes-version 1.14.1
    ```

8.  Once completed, download and install kubectl (Client for k8s)

    ```console
    $ az aks install-cli
    ```

9.  Download the aks credentials

    ```console
    $ az aks get-credentials -n aks1020 -g mixed-aks-group
    ```

10. Now, list all nodes in cluster

    ```console
    $ kubectl get nodes -o wide
    ```

11. Installing Helm CLI and Tiller.

    Download and Install helm on Ubuntu 16+ System
    ```console
    $ sudo snap install helm --classic
    ```

    Download and Install helm on Windows 10 PRO System (Use powershell)
    Option 1: Using wget on powershell
    ```
    $ wget https://get.helm.sh/helm-v2.14.3-windows-amd64.zip -O helm.zip
    
    ```

    Option 2: Using Invoke-WebRequest on powershell
    ```console
    $ Invoke-WebRequest -UseBasicParsing -Uri https://get.helm.sh/helm-v2.14.3-windows-amd64.zip -OutFile helm.zip
    
    ```

    Extract the contents of downloaded ZIP into new helm home directory (Replace `mahendra` with your username)
    ```console
    $ mkdir c:\users\mahendra\.helm
    $ $ expand-archive -Path ./helm.zip -DestinationPath C:\Users\mahendra\.helm
    ```

12. Initialize helm cli and tiller (On Cluster)
    
    > NOTE: tiller container is LINUX based, to avoid deploying on windows node, use node-selector constraint while deploying tiller.

    ```console
    $ helm init --node-selectors "beta.kubernetes.io/os"="linux"
    ```

13. Install RBAC role and user for Tiller (To avoid error: `No available release name found`)

    ```console
    $ kubectl create serviceaccount --namespace kube-system tiller
    $ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
    $ kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

    ```
    > 