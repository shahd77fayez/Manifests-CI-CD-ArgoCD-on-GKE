# Manifests-CI-CD-ArgoCD-on-GKE
## Objective
Deploy a complete GitOps-driven CI/CD pipeline using Jenkins and ArgoCD on a Google Kubernetes Engine (GKE) cluster. This includes infrastructure provisioning,    containerization, artifact management, and automated deployment using ArgoCD. 

---
## 📁 Project Repositories

- **Repo 1 – Application Code**
  - Contains the sample Node.js app
  - Includes:
    - `Dockerfile`
    - `Jenkinsfile`

- **Repo 2 – Manifests / Helm Chart**
  - Contains:
    - Helm charts
---

## ✅ Prerequisites

- Google Cloud CLI (`gcloud`) OR MiniKube
- Kubernetes CLI (`kubectl`)
- Helm CLI (`helm`)
- Docker CLI
- Jenkins installed (local, GKE, or external)
- DockerHub Account (or any container registry)
- ArgoCD CLI (`argocd`)

---

## Step-by-Step Tasks

## 🏗️ Step 1: Provision GKE Infrastructure OR MiniKube

1. **Create GKE Cluster**:
   ```bash
    gcloud container clusters create gitops-gke
     --zone us-central1-a
     --num-nodes 1
   ```
- **MiniKube**:
    ```bash
        Minikube Start
    ```
---
## 🎯 Step 2: Deploy ArgoCD & ArgoCD Image Updater
1. **Install ArgoCD**:
    ```bash
    kubectl create namespace argocd 
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```
2. **Install ArgoCD Image Updater**:
    ```bash
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
    ```
    ![image alt](https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE/blob/bf05714eb79df2521290d30416efb68a73aa8890/Images/argocd%20install%20and%20updater.png) 
2. **Expose ArgoCD Server**:
    ```bash
    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
    minikube service list
    ```
3. **Access ArgoCD Web UI**:
    - get initial password
    ```bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```
    - open argo UI
    ```bash
        http://192.168.49.2:32043
    ```
     ![ image alt](https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE/blob/b7861a23136340a18e2c836795524da0b605c915/Images/Argo%20cd%20url%20aceess.png) 
---
## 📁 Step 3: Create & Organize GitHub Repositories

- **Repo #1 - Application Repo**:
    - DockerFile
    - JenkinsFile
    - Github Repo Link
        ```bash
        https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE.git
        ```

- **Repo #2 - Helm/Manifests Repo**:
    - Charts
    - Github Repo Link
        ```bash
        https://github.com/shahd77fayez/Manifests-CI-CD-ArgoCD-on-GKE.git
--- 

## ⚙️ Step 4: Implement CI with Jenkins

1. **Run Jenkins Container**:
    ```bash
    docker run -d --name jenkins --user root --privileged --network jenkins --network-alias docker --env DOCKER_TLS_CERTDIR= -v /var/run/docker.sock:/var/run/docker.sock -v jenkins-data:/var/jenkins_home -p 9090:8080 -p 50000:50000 jenkins/jenkins:lts
    ```
2. **Create Jenkins Pipeline Job**:
    - Pipeline script from SCM → Link to App Repo
        ![image alt](https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE/blob/b7861a23136340a18e2c836795524da0b605c915/Images/Jenkins%20SCM%20to%20link%20repo.png)
    - Run Job:
        ![image alt](https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE/blob/b7861a23136340a18e2c836795524da0b605c915/Images/jenkins%20build%20success.png)
---
## 🔁 Step 5: Configure ArgoCD for CD
1. **create application on ArgoCd**:
    ![image alt](https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE/blob/b7861a23136340a18e2c836795524da0b605c915/Images/application%20argo.png)
---
## 🔄 Step 6: Apply Argo CD Image Updater
1. **Configure API access token secret**:
    ```bash
    kubectl create secret generic git-creds -n argocd --from-literal=username=<my-username> --from-literal=password=<PAT-Github>
    ```
    ![image alt](https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE/blob/b7861a23136340a18e2c836795524da0b605c915/Images/secrets%20of%20updater.png)
2. **Configure Image Updater annotations**:
    ```bash
        annotations:
            argocd-image-updater.argoproj.io/image-list: shahdfayez06/simple-nodeapp
            argocd-image-updater.argoproj.io/simple-node-app.update-strategy: latest
            argocd-image-updater.argoproj.io/write-back-method: git:secret:argo/git-creds
    ```
---
## 🧪 Step 7: Test the Pipeline
1. **Modify app code & pipeline SCM run push image docker hub**:
   ![image alt](https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE/blob/42ff2953b6e86567cd885fd3cb66eaa0ba7b0eee/Images/docker%20hub%20new%20image%20tag.png)
2. image updater observe
    ![image alt](https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE/blob/b7861a23136340a18e2c836795524da0b605c915/Images/image%20updater%20sync.png)
3. **show argo cd health status of app**:
    ![image alt](https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE/blob/b7861a23136340a18e2c836795524da0b605c915/Images/argo%20cd%20healthy%20app.png)
4. **application acess**:
    - get url for app
    ```bash
        minikube service list
    ```
    ![image alt](https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE/blob/b7861a23136340a18e2c836795524da0b605c915/Images/service%20list%20for%20nodeapp.png)

    - access it
    ![image alt](https://github.com/shahd77fayez/CI-CD--Jenkins--ArgoCD-on-GKE/blob/b7861a23136340a18e2c836795524da0b605c915/Images/running%20app.png)

    

