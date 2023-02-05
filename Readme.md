    Deploy an AKS cluster using the Azure CLI.
    Run a sample multi-container application with a web front-end and a Redis instance in the cluster.


    Prerequisites
    Create a resource group
    Create AKS cluster
    Connect to the cluster
    Deploy the application
    Test the application
    Delete the cluster
    Next steps

DeployVoteapplicationinAKS
This manifest includes two Kubernetes deployments:
    The sample Azure Vote Python applications.
    A Redis instance
Start command in CLI:
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
az group create --name myResourceGroup --location eastus
az aks create -g myResourceGroup -n myAKSCluster --enable-managed-identity --node-count 1 --enable-addons monitoring --enable-msi-auth-for-monitoring  --generate-ssh-keys
az aks disable-addons -a monitoring -n myAKSCluster -g myResourceGroup
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
kubectl get nodes
nano azure-vote.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-back
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-back
  template:
    metadata:
      labels:
        app: azure-vote-back
    spec:
      nodeSelector:
        "kubernetes.io/os": linux
      containers:
      - name: azure-vote-back
        image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
        env:
        - name: ALLOW_EMPTY_PASSWORD
          value: "yes"
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 6379
          name: redis
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-back
spec:
  ports:
  - port: 6379
  selector:
    app: azure-vote-back
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-front
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-front
  template:
    metadata:
      labels:
        app: azure-vote-front
    spec:
      nodeSelector:
        "kubernetes.io/os": linux
      containers:
      - name: azure-vote-front
        image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
        env:
        - name: REDIS
          value: "azure-vote-back"
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
 ------------------------ 
 kubectl apply -f azure-vote.yaml
 kubectl get service azure-vote-front --watch
 
 Now, we need to push into git
 git init
 git add azure-vote.yaml
 git status
 git commit -m "1st commit"
 git branch -M main
 git remote add origin git@github.com:sundeepbhatia1989/DeployVoteapplicationinAKS.git
 git push -u origin main
 --------------------
 Ref: https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-cli
 AKS learning: https://learn.microsoft.com/en-us/azure/aks/concepts-clusters-workloads#deployments-and-yaml-manifests
 AKS Service account info: https://learn.microsoft.com/en-us/samples/azure/azure-quickstart-templates/aks/
 How to clear azure cli history?
        Before I show how to set up the crontab job for this, know that the ~/.bash_history file can be cleared with the command:
    Command:
        cat /dev/null > ~/.bash_history
