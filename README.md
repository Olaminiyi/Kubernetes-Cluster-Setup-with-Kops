# KUBERNETES SETUP WITH KUBERNETES OPERATIONS (KOPS).

**Kops** - **"Kubernetes Operations"**, is a command-line tool used for deploying, managing, and operating Kubernetes clusters on various cloud platforms. Kubernetes is an open-source container orchestration platform that helps manage the deployment, scaling, and operation of containerized applications.

Kops specifically focuses on simplifying the process of setting up and managing Kubernetes clusters on cloud infrastructure providers like Amazon Web Services (AWS), Google Cloud Platform (GCP), and others. It allows users to define and customize their Kubernetes clusters using configuration files, which makes it easier to maintain consistent and reproducible cluster configurations.

**Key features of Kops include:**

**Cluster Provisioning:** Kops helps automate the provisioning of infrastructure resources required for a Kubernetes cluster, such as virtual machines, networking, and storage.

**Cluster Upgrades:** Kops simplifies the process of upgrading Kubernetes clusters to newer versions, ensuring that the update process is seamless and minimizes downtime.

**Scaling:** Kops allows you to scale your cluster by adding or removing nodes, adjusting the resources allocated to nodes, and managing node groups.

**High Availability:** Kops supports configuring high availability features of Kubernetes, such as distributing master nodes across multiple availability zones, which enhances the resiliency of the cluster.

**Customization:** Users can define various cluster configurations, including node instance types, networking settings, add-ons, and other parameters, according to their requirements.

**Validation and Verification:** Kops includes tools for verifying the correctness of your cluster configuration and diagnosing potential issues before deploying the cluster.

In this instance, I will be configuring a Kubernetes cluster by employing the Kops tool.

**Prerequisites:**

- A domain name for Kubernetes DNS record - olami.uk.
- AWS account

**Task:**

- Log into AWS account:
    - Create an ubuntu Instance.
    - Create S3 bucket.
    - Create an IAM user for awscli.
    - Create Route 53 hosted zone.

- SSH into the instance

- Setup the following:
    - kops
    - kubectl
    - awscli
    - ssh keys

Purchase a domain name from a domain name provider. eg godaddy.com and namesilo.com etc.

Create an ubuntu EC2 instance for Kops.

![alt text](images/1.1.png)

**Create S3 bucket**

![alt text](images/1.2.png)

Kops will establish a cluster and utilize awscli for interaction with AWS services. To enable access to these services via awscli commands, valid credentials are essential.

Create an IAM role that will be endowed with administrator-level privileges for the kops instance. This IAM role is essential as it will interact with a wide range of services, including S3 buckets and Route53. Once this role is in place, the next step involves setting up the kops instance to utilize the associated access keys. I will named the user 'terraform'

![alt text](images/1.3.png)

**Username: terraform**

![alt text](images/1.4.png)

Give Admnistrator access > create user

![alt text](images/1.5.png)

![alt text](images/1.6.png)

![alt text](images/1.7.png)

**Click on security credentials > Access Key > Command Line Interface (CLI) > create**

![alt text](images/1.8.png)
![alt text](images/1.9.png)
![alt text](images/1.10.png)
![alt text](images/1.11.png)

**The next thing is to configure our aws with security credentials we created**
- first make sure you have aws CLI install, if not [download and install](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- go to the command prompt

```
aws config
```
enter the Access key and Secret Access when prompted

![alt text](images/1.12.png)

- check if the profile has been set by your configuration.
- type the command below on the command prompt.

```
aws sts get-caller-identity
```
![alt text](images/1.13.png)

**Get a domain name** 

**Create Hosted zone in Route53**

![alt text](images/1.14.png)

![alt text](images/1.15.png)

This creates the ns servers url which we will add to the domain servers register in the DNS provider [123-Reg](https://www.123-reg.co.uk/).

Log into your DNS provider account and update the name servers with the ns server url that was created earlier.

![alt text](images/1.16.png)

![alt text](images/1.17.png)

SSH into the EC2 instance and complete the setup.

Generate ssh keys with this command
```
ssh-keygen
```
![alt text](images/1.18.png)

Install awscli
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

![alt text](images/1.19.png)

Configure it to use the access keys of the IAM user that was created.
```
aws configure
```
![alt text](images/1.20.png)

Setup kubectl and kops

To setup kubectl we will refer to the [kubernetes documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux).

Install kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
Make kubectl executable
```
sudo chmod +x kubectl
```
To make kubectl globally accessible
```
sudo mv kubectl /usr/local/bin/
```
Verify
```
kubectl version --client
```
![alt text](images/1.21.png)

To setup kops we will refer to the [kops documentation](https://github.com/kubernetes/kops/releases) for various versions og kops.

I will be installing version 1.29.0

![alt text](images/1.22.png)
![alt text](images/1.22.png)

Copy the above link and download the binary
```
wget https://github.com/kubernetes/kops/releases/download/v1.29.0/kops-linux-amd64
```
Make kops executable
```
chmod +x kops-linux-amd64
```
To make kops globally accessible
```
sudo mv kops-linux-amd64 /usr/local/bin/kops
```
```
kops version
```
![alt text](images/1.24.png)

Verify domain kubekops.olami.uk
```
nslookup -type=ns kubekops.olami.uk
```
![alt text](images/1.25.png)

Create configurations for the cluster and store in the S3 bucket using the command:
```
$ kops create cluster --name=kubekops.olami.uk --state=s3://project-kops-states --zones=us-east-1a,us-east-1b --node-count=2 --node-size=t3.small --master-size=t3.medium --dns-zone=kubekops.olami.uk --node-volume-size=8 --master-volume-size=8
```

![alt text](images/1.26.png)


Make sure to specify the `--node-volume-size` and `--master-volume-size` if not it will create huge volume for the etcd.

Configure and create the cluster by running the command
```
kops update cluster --state=s3://project-kops-states --name kubekops.olami.uk --yes --admin
```

![alt text](images/1.27.png)

![alt text](images/1.28.png)

Wait for 15 minutes for the cluster to create.

On every execution of the **'kops'** command, inclusion of the associated bucket name is mandatory. In the absence of this S3 bucker name the command will not work.

Validate the cluster
```
kops validate cluster --state=s3://project-kops-states
```
![alt text](images/1.29.png)


Upon the installation of kops, a configuration file named **.kube/config** was generated within the home directory for the purpose of executing **kubectl** commands.
```
cat .kube/config
```
![alt text](images/1.30.png)

To see the nodes we run the command
```
kubectl get nodes
```
![alt text](images/1.31.png)

To delete the cluster
```
kops delete cluster --name=kubekops.olami.uk --state=s3://project-kops-states --yes
```
