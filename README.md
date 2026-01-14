# Monitor NGINX Pod Logs on Datadog
This guide shows how to correctly install Datadog on Amazon EKS and collect NGINX pod logs, without common installation errors.

---

# Prerequisites
Make sure you have:

- EC2  (c7i-flex.large)
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
### Write Pod.Yaml File
```sh
nano pod.yaml
```
```sh
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```
### Apply Pod.Yaml
```sh
kubectl apply -f pod.yaml
```
### Write Svc.Yaml
```sh
nano svc.yaml
```
```sh
apiVersion: v1
kind: Service
metadata:
  name: my-lb-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```
### Apply Svc.Yaml
```sh
kubectl apply -f svc.yaml
```
### Check Pods
```sh
kubectl get pods
```
### Check Svc
```sh
kubectl get svc
```

### Helm Installation
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
### verify
```sh
helm version
```
###  Create Datadog Namespace & Secrets
```sh
kubectl create namespace datadog
```
###  Create API Key secret
```sh
kubectl create secret generic datadog-secret \
  --from-literal=api-key=<YOUR_DATADOG_API_KEY> \
  -n datadog
```

### Create Application Key secret (REQUIRED)
 This step is mandatory.
Missing this causes CreateContainerConfigError in datadog-cluster-agent.
```sh
kubectl create secret generic datadog-app-secret \
  --from-literal=app-key=<YOUR_DATADOG_APP_KEY> \
  -n datadog
```

### Add Datadog Helm Repository
```sh
helm repo add datadog https://helm.datadoghq.com
helm repo update
```
### Create Helm Values File (datadog-values.yaml)
 This file is clean, non-duplicated, and non-deprecated.
```yaml
datadog:
  apiKeyExistingSecret: datadog-secret
  appKeyExistingSecret: datadog-app-secret
  site: datadoghq.com            # change only if EU or US3
  clusterName: my-eks-cluster

  logs:
    enabled: true
    containerCollectAll: true

  apm:
    portEnabled: true

  processAgent:
    enabled: true
    processCollectionEnabled: true

  orchestratorExplorer:
    enabled: true

clusterAgent:
  enabled: true
  replicas: 2
  pdb:
    create: true

agent:
  containerLogs:
    enabled: true
```

### Install Datadog Using Helm
```sh
helm upgrade --install datadog-agent datadog/datadog \
  -f datadog-values.yaml \
  -n datadog
```
Restart agents to apply config:
```sh
kubectl rollout restart daemonset datadog-agent -n datadog
kubectl rollout restart deployment datadog-agent-cluster-agent -n datadog
```
### Verify Datadog Installation
```
kubectl get pods -n datadog
```
Expected output:
```sql
datadog-agent-xxxxx                2/2 Running
datadog-agent-cluster-agent        1/1 Running
datadog-agent-operator             1/1 Running
```

If you see CreateContainerConfigError, re-check:

- datadog-app-secret
- key name is exactly app-key
- secret exists in datadog namespace

---
# Datadog - Infrastructure - kubernetes overview 

---
### Cleanup (Optional)
```sh
helm uninstall datadog-agent -n datadog
kubectl delete deployment nginx
kubectl delete service nginx
kubectl delete namespace datadog
```
