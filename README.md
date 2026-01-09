# Monitor NGINX Pod Logs on Datadog
This guide shows how to correctly install Datadog on Amazon EKS and collect NGINX pod logs, without common installation errors.

---

# Prerequisites
Make sure you have:

- EC2 
- Create EKS cluster
    - Add Eks Nodes 
- Aws configure
- Helm v3+
- A Datadog account
- Datadog API Key
- Datadog Application Key
- Datadog site:
   - datadoghq.com → https://app.datadoghq.com
   - datadoghq.eu → https://app.datadoghq.eu
   - us3.datadoghq.com → https://app.us3.datadoghq.com
  Important:
The Datadog UI URL (app.datadoghq.com) is NOT the value used in Helm.
Always use datadoghq.com, datadoghq.eu, or us3.datadoghq.com.
---

##  Installation Steps & Commands
###  Root 
~~~sh
sudo -i
~~~
###  Update 
~~~sh
apt update
~~~

###  Install kubectl 
#### Download the latest release with the command
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

#### Validate the binary
```sh
 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```

#### Validate the kubectl binary against the checksum file 
```sh
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```

#### Install kubectl
```sh
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

#### Note: If you do not have root access on the target system, you can still install kubectl to the ~/.local/bin directory:
```sh
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
```
```sh
kubectl version --client
```

###  Install AWS CLI on Ubuntu
```sh
snap install aws-cli --classic
```

###  Configure AWS CLI

#### To connect AWS using CLI we have configure AWS user using below command
```sh
aws configure
```

### Log In Into EKS cluster
```sh
aws eks update-kubeconfig --name (**EKS Cluster Name**)
```
Ex. aws eks update-kubeconfig --name k8s

### Check Cluster Information
```sh
kubectl cluster-info
```
