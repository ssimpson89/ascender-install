The Ascender installer is a script that makes for relatively easy
install of Ascender Automation Platform on Kubernetes platforms of
multiple flavors. The installer is being expanded to new Kubernetes
platforms as users/contributors allow, and if you have specific needs
for a platform not yet supported, please submit an issue to this
Github repository.

## Table of Contents

- [General Prerequisites](#general-prerequisites)
- [EKS-specific Prerequisites](#eks-specific-prerequisites)
- [Install Instructions](#install-instructions)

## General Prerequisites

If you have not done so already, be sure to follow the general
prerequisites found in the [Ascender-Install main
README](../../README.md#general-prerequisites)

## AKS-specific Prerequisites

### AKS User, policy and tool requirements
- The Ascender installer for AKS requires installation of the [Azure Commmand Line Interface](https://learn.microsoft.com/en-us/cli/azure/) before it is invoked. Instructions for the Linux installer can be found at [this link](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux).
  - Once the Azure Command Line Interface is installed, run the following command to set the active Azure user to one with the appropriate permissions to run the Ascender installer on AKS: `$ az login`. 
    - If the Azure CLI can open your default browser, it initiates authorization code flow and opens the default browser to load an Azure sign-in page.
    - Otherwise, it initiates the [device code flow](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-device-code) and instructs you to open a browser page at [https://aka.ms/devicelogin](https://aka.ms/devicelogin). Then, enter the code displayed in your terminal.
    - If no web browser is available or the web browser fails to open, you may force device code flow with az login --use-device-code.

## Install Instructions

### Obtain the sources

You can use the `git` command to clone the ascender-install repository or you can download the zipped archive. 

To use git to clone the repository run:

```
git clone https://github.com/ctrliq/ascender-install.git
```
This will create a directory named `ascender-install` in your present working directory (PWD).

We will refer to this directory as the `<repository root>` in the remainder of this instructions.

### Set the configuration variables for an eks Install

#### inventory file

You can copy the contents of [aks.inventory](./aks.inventory) in this directory, to `<repository root>`/inventory.

#### custom.config.yml file

You can run the bash script at 

```
<repository root>>/config_vars.sh
```

The script will take you through a series of questions, that will populate the variables file requires to install Ascender. This variables file will be located at:

```
<repository root>/custom.config.yml
```

Afterward, you can simply edit this file should you not want to run the script again before installing Ascender.

The following variables will be present after running the script:

- `k8s_platform`: This variable specificies which Kubernetes platform Ascender and its components will be installed on.
- `k8s_protocol`: Determines whether to use HTTP or HTTPS for Ascender and Ledger.
- `USE_AZURE_DNS`: Determines whether to use Route53's Domain Management, or a third-party service such as Cloudflare, or GoDaddy. If this value is set to false, you will have to manually set a CNAME record for `ASCENDER_HOSTNAME` and `LEDGER_HOSTNAME` to point to the AWS Loadbalancers that the installer creates.
- `ASCENDER_HOSTNAME`: The DNS resolvable hostname for Ascender service.
- `LEDGER_HOSTNAME`: The DNS resolvable hostname for Ascender service.
- `ASCENDER_DOMAIN`: The Hosted Zone/Domain for all Ascender components. 
  - this is a SINGLE domain for both Ascender AND Ledger.
- `EKS_CLUSTER_NAME`: The name of the eks cluster on which Ascender will be installed. This can be an existing eks cluster, or the name of the one to create, should the `eksctl` tool not find this name amongst its existing clusters.
- `EKS_CLUSTER_REGION`: The AWS region hosting the eks cluster
- `EKS_CLUSTER_CIDR`: The eks cluster subnet in CIDR notation
- `EKS_K8S_VERSION`: The kubernetes version for the eks cluster; available kubernetes versions can be found [here](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html)
- `EKS_INSTANCE_TYPE`: The worker node instance type. 
- `EKS_MIN_WORKER_NODES`: The minimum number of worker nodes that the cluster will run
- `EKS_MAX_WORKER_NODES`: The maximum number of worker nodes that the cluster will run
- `EKS_NUM_WORKER_NODES`: The desired number of worker nodes for the eks cluster
- `EKS_WORKER_VOLUME_SIZE`: The size of the Elastic Block Storage volume for each worker node
- `EKS_SSL_CERT`: The ARN for the SSL certificate; required when k8s_lb_protocol is https. The same certificate is used for all components (currently Ascender and Ledger); as such, we recommend that the certificate is set for a wildcard domain, (e.g., *.example.com).


USE_AZURE_DNS: Determines whether to use Azure DNS Domain Management, or a third-party service such as Cloudflare, or GoDaddy. If this value is set to false, you will have to manually set a CNAME record for `ASCENDER_HOSTNAME` and `LEDGER_HOSTNAME` to point to the AWS Loadbalancers that the installer creates.
# The name of the eks cluster to install Ascender on - if it does not already exist, the installer can set it up
AKS_CLUSTER_NAME: ascender-aks-cluster
# Specifies whether the AKS cluster needs to be provisioned (provision), exists but needs to be configured to support Ascender (configure), or exists and needs nothing done before installing Ascender (no_action)
AKS_CLUSTER_STATUS: provision
# The Azure region hosting the aks cluster
AKS_CLUSTER_REGION: eastus
# The kubernetes version for the aks cluster; available kubernetes versions can be found here:
AKS_K8S_VERSION: "1.29"
# The aks worker node instance types
AKS_INSTANCE_TYPE: "Standard_D2_v2"
# The desired number of aks worker nodes
AKS_NUM_WORKER_NODES: 3
# The volume size of aks worker nodes in GB
AKS_WORKER_VOLUME_SIZE: 100
# TLS Certificate file location on the local installing machine
tls_crt_path: "/home/rocky/ascender.crt"
# TLS Private Key file location on the local installing machine
tls_key_path: "/home/rocky/ascender.key"

### Run the setup script

To begin the setup process, from the <repository root> directory in this repository, type:

```
sudo <repository root>/setup.sh
```

Once the setup is completed successfully, you should see a final output similar to:

```
...<OUTPUT TRUNCATED>...
PLAY RECAP *************************************************************************************************************************
ascender_host              : ok=14   changed=6    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
localhost                  : ok=72   changed=27   unreachable=0    failed=0    skipped=4    rescued=0    ignored=0

ASCENDER SUCCESSFULLY SETUP
```


### Connecting to Ascender Web UI

You can connect to the Ascender UI at https://`ASCENDER_HOST`

The username is and the corresponding password is stored in `<repository root>`/custom.config.yml under the `ASCENDER_ADMIN_USER` and `ASCENDER_ADMIN_PASSWORD` variables, respectively.


## Uninstall Instructions

After running `setup.sh`, `tmp_dir` will contain timestamped kubernetes manifests for:

- `ascender-deployment-{{ k8s_platform }}.yml`
- `ledger-{{ k8s_platform }}.yml` (if you installed Ledger)
- `kustomization.yml`
- `eks-cluster-manifest.yml` (if you provisioned a new EKS cluster with the Ascender installer)

Remove the timestamp from the filename and then run the following
commands from within `tmp_dir``:

- `$ kubectl delete -f ascender-deployment-{{ k8s_platform }}.yml`
- `$ kubectl delete -f ledger-{{ k8s_platform }}.yml`
- `$ kubectl delete -k .`

To delete an EKS cluster created with the Ascender installer, run the following command from within the `tmp_dir`

- `$ eksctl delete cluster -f eks-cluster-manifest.yml --disable-nodegroup-eviction --force`