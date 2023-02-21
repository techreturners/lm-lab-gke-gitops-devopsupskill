# GKE and Terraform

The steps here cover how to provision your GKE cluster and container registry.

## Instructions

**NOTE:** You might have already performed some of the steps (such as installing tools) mentioned below. If so you can ignore those steps and move directly to **Step 7**

### Step 1 - Fork and clone

Fork this repository into your own GitHub account and then clone (your forked version) down to your local machine.

### Step 2 - Install and configure the GCloud SDK

We'll use the Google Command Line tool to get information from the cluster.

If you haven't already installed it then you can follow below to install.

To install this you can install manually by following [these instructions](https://cloud.google.com/sdk/docs/quickstarts) or if you have a package manager you can use the instructions below:

**Homebrew**

```
brew install --cask google-cloud-sdk
```

**Chocolatey**

```
choco install gcloudsdk
```

Once it is installed we need to configure it to point at your Google Cloud Project - to do this run:

```
gcloud init
```

When prompted ensure you pick the correct Google Cloud Project.

### Step 3 - Install kubectl tool

The `kubectl` tool will be utilised to interact with your Kubernetes (K8S) cluster.

You can install this manually by following this guide below:

https://kubernetes.io/docs/tasks/tools/install-kubectl/

Or alternatively if you use a package manager it can be installed using the package manager:

**Homebrew**

```
brew install kubernetes-cli
```

**Chocolatey**

```
choco install kubernetes-cli
```

You can verify that it has installed correctly by running 

```
kubectl version --client --output=yaml
```

It should print something like the below:

```
clientVersion:
  buildDate: "2023-01-18T15:51:24Z"
  compiler: gc
  gitCommit: 8f94681cd294aa8cfd3407b8191f6c70214973a4
  gitTreeState: clean
  gitVersion: v1.26.1
  goVersion: go1.19.5
  major: "1"
  minor: "26"
  platform: darwin/arm64
kustomizeVersion: v4.5.7
```

### Step 4 - Ensure kubectl can connect to cluster

Recent changes to open source Kubernetes has meant that there are tweaks to be made to ensure you can connect to GKE from your laptop. This means we need to install some extra features from the Google command line. To install authentication plugin run:

```
gcloud components install gke-gcloud-auth-plugin
```

To verify the installtion, if you are on a Mac/Linux you can run:

```
gke-gcloud-auth-plugin --version
```

To verify the installtion, if you are on Windows you can run:

```
gke-gcloud-auth-plugin.exe --version
```

> **⚠️Note: GKE Auth**
> For those interested, you can read a bit more about the [changes here](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke)

### Step 5 - Explore the files

Before we go ahead and create your cluster its worth exploring the files.

Oh and before we do explore, the files in this directory could have been named whatever we like. For example the **outputs.tf** file can have been called **foo.tf** - we just chose to call it that because it contained outputs. So the naming was more of a standard than a requirement.

**variables.tf**

This file contains the definitions of all the terraform [input variables](https://www.terraform.io/language/values/variables) we can set to drive your terraform execution.

**terraform.tfvars**

Think of this as the place where you define the actual values of variables to be used by your terraform configuration.

**vpc.tf**

This file contains the setup for the Virtual Private Cloud (VPC - remember those from session two) that your GKE will be placed into. Take a look into it and spot those CIDR ranges. Also remember the IP address ranges falling within that internal network designation from [RFC 1918](https://www.rfc-editor.org/rfc/rfc1918)

**gke.tf** 

Does the actual job of provisioning a GKE cluster and a separately managed node pool (recommended).

You'll also see that this file contains a variable called `gke_num_nodes` that is defaulted to 1. This means that one node will be created in each subnet. Totaling 3 nodes for your cluster.

**outputs.tf**

This file defines the outputs that will be produced by terraform when things have been provisioned.

**versions.tf** 

Configures the terraform providers (in our case the GCP provider) and sets the Terraform version to at least 1.3.0.

### Step 6 - Update the tfvars file

Now you know the files the next step is to update the tfvars file according to your project.

Update the project ID to be that of your Google Project ID and change the region if you would like it to be anything other than the UK.

### Step 7 - Update service account

In the terraform installation video you setup a service account. You'll need to re-use that service account again in order for Terraform to authenticate with your Google cloud account.

First **copy** the service account to this directory.

Then rename that service account file to be called **service-account.json** 

The reason for the rename is because you'll see it [mentioned in the **versions.tf**](./versions.tf) file and also in the **.gitignore** file. Choosing the name **service-account.json** will ensure you don't accidentally commit it due to us adding it to the gitignore file.

### Step 8 - Initialise terraform

We need to get terraform to pull down the [Terraform Google provider](https://registry.terraform.io/providers/hashicorp/google/latest).

In order to do this run:

```
terraform init
```

You should see something similar to the below:

```
Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/google versions matching "3.58.0"...
- Installing hashicorp/google v3.58.0...
- Installed hashicorp/google v3.58.0 (signed by HashiCorp)
```

### Step 9 - Review changes with a plan

Firstly run a **plan** to see if what Terraform decides will happen.

```
terraform plan
```

### Step 10 - Create your cluster with apply

We can then create your cluster by applying the configuration.

**NOTE** If you find it complain that an API isn't enabled. [Compute Engine API](https://console.developers.google.com/apis/api/compute.googleapis.com/overview) and [Kubernetes Engine API](https://console.cloud.google.com/apis/api/container.googleapis.com/overview) are required for terraform apply to work on this configuration. Enable both APIs for your Google Cloud and then continue.

```
terraform apply
```

Sit back and relax - it might take 10 mins or so to create your cluster.

Once its finished it'll output something like the info below. Those **outputs** are defined in the **outputs.tf** file.

```
Outputs:

kubernetes_cluster_host = "35.246.22.132"
kubernetes_cluster_name = "devops-upskill-305410-gke"
project_id = "devops-upskill-123456"
region = "europe-west2"
```

Once its done you'll have your Kubernetes cluster all ready to go!!!

### Step 11 - Configure your **kube control** 

[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) is used to issue actions on your cluster.

We need to configure **kubectl** to be able to authenticate with your cluster.

Firstly if you remember step 4 we installed the GKE auth plugin. We need to set an environment variable that tells kubectl that we wish to use the auth plugin. To do this:

```
export USE_GKE_GCLOUD_AUTH_PLUGIN=True
```

Then you will use the Google Cloud Command Line to get the credentials. Notice how we reference the terraform outputs in the command below:


```
gcloud container clusters get-credentials $(terraform output -raw kubernetes_cluster_name) --region $(terraform output -raw region) --project $(terraform output -raw project_id)
```

It should say something like:

```
Fetching cluster endpoint and auth data.
Kubernetes v1.21.0-alpha+eabdbc0a40fe7efda92e10270f27b0a3485fb743
kubeconfig entry generated for devops-upskill-305410-gke.
```

### Step 12 - Check if kubectl can access cluster

You can now verify if `kubectl` can access your cluster.

Go ahead and see how many nodes are running:

```
kubectl get nodes
```

It should show something like:

```
NAME                                                  STATUS   ROLES    AGE   VERSION
gke-devops-upskill-3-devops-upskill-3-6ab12de4-6p9p   Ready    <none>   18m   v1.18.12-gke.1210
gke-devops-upskill-3-devops-upskill-3-eada08f1-793v   Ready    <none>   18m   v1.18.12-gke.1210
gke-devops-upskill-3-devops-upskill-3-fa0c8ee0-j7qt   Ready    <none>   18m   v1.18.12-gke.1210
```

Exciting eh!!! 🚀

Now you can head back over to the [README](../README.md) for the next stage which is pushing your docker images to your registry.