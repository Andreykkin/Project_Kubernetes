
# 🔧 Prerequisites
- Kubernetes cluster (v1.19+)
- ArgoCD installed (for GitOps deployment)
- Storage provider available (for PV provisioning)
- kubectl configured with cluster access

# 🚀 Features
- Full-stack Wiki Application: Ready-to-deploy wiki solution for Kubernetes environments
- ArgoCD Integration: Pre-configured Application manifests for GitOps workflows
- PostgreSQL with Persistent Storage: Complete storage configuration including:

    - PersistentVolume (PV) definitions

    - PersistentVolumeClaim (PVC) templates

    - StorageClass configurations

- Manual Deployment Ready: All YAML files provided for direct kubectl apply operations
# 🎯 Use Cases

-    Learning GitOps: Example of application deployment using ArgoCD patterns

-    Persistent Storage Demos: Reference implementation for stateful applications

-    On-premises Kubernetes: Manual storage provisioning for environments without dynamic provisioning

-    ArgoCD Testing: Ready-to-use application for testing ArgoCD installations
# Installation
### First step: Clone git repo:
```bash
git clone https://github.com/Andreykkin/Project_Kubernetes.git
cd Project_Kubernetes
```
### Step Two: Create Persistent Storage for database:
```bash
kubectl create namespace production
kubectl apply -f Database/storageclass.yaml ;
kubectl apply -f Database/persistent_volume.yaml ;
kubectl apply -f Database/persistent_volume_claim.yaml -n production
```
### Step three: Deploy Database and configure for wiki 
#### Deploy helm with your password
```bash
helm install postgresql oci://registry-1.docker.io/bitnamicharts/postgresql --set primary.persistence.existingClaim=pvc-for-postgresql,auth.postgresPassword=<YOUR-PASSWORD> --namespace production
```
#### Then log into DB:
```bash
export POSTGRES_PASSWORD=$(kubectl get secret --namespace production postgresql -o jsonpath="{.data.postgres-password}" | base64 -d) ;
kubectl run postgresql-client --rm --tty -i --restart='Never' -n production --image bitnami/postgresql:latest --env="PGPASSWORD=$password" --command -- psql --host postgresql -U postgres -d postgres -p 5432
```
#### Create DB, wiki user, grant permissions for wiki:
```bash
CREATE DATABASE wiki;
CREATE USER wiki WITH ENCRYPTED PASSWORD 'your-password-for-wiki';
ALTER DATABASE wiki OWNER TO wiki;
GRANT ALL PRIVILEGES ON DATABASE wiki TO wiki;
GRANT ALL PRIVILEGES ON SCHEMA public TO wiki;
```
### Step four: Deploy wiki application to ArgoCD
#### Deploy secret with wiki postgres password
```bash
kubectl create secret generic wiki-postgresql-password \
  --from-literal=postgresql-password='<wiki-user-password>' \:
  -n production
```
#### Use ArgoCD UI or kubectl
##### Kubectl way
```bash
kubectl apply -n argocd -f wiki.js/wiki_app.yaml
```
## Congratulations! You deployed it!
