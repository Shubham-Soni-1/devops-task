# CI/CD Pipeline for Node.js App Deployment

This repository contains a Jenkins pipeline (`Jenkinsfile`) and supporting files to automate building, packaging, and deploying a Node.js application (`logo-server`) onto an EC2 instance.

---

## 🔑 Jenkins Credentials Setup

Before running the pipeline, you must configure credentials in Jenkins:

### 1. DockerHub Credentials
- Navigate to: **Dashboard → Manage Jenkins → Credentials → Add Credentials**
- Select: **Username with password**
- Enter:
  - **Username** → Your DockerHub username
  - **Password** → Your DockerHub password or Personal Access Token
  - **ID** → `DOCKERHUB_CREDS`
  - **Description** → `DockerHub credentials`

### 2. EC2 SSH Key
- Navigate to: **Dashboard → Manage Jenkins → Credentials → Add Credentials**
- Select: **SSH Username with private key**
- Enter:
  - **Username** → EC2 instance username (`ec2-user`, `ubuntu`, etc.)
  - **Private Key** → Paste the contents of your `.pem` key from AWS
  - **ID** → `EC2_SSH_KEY`
  - **Description** → `EC2 SSH Private Key`

---

## 🚀 Current Flow (Docker-based Deployment)

1. **Source Checkout**  
   Jenkins pulls the application source code from GitHub.

2. **Build & Test**  
   - Installs dependencies (`npm install`)  
   - Runs basic checks (`node -v`)  

3. **Docker Image Build**  
   Builds a Docker image:  
   shubhamsoni3332/logo-server:<BUILD_NUMBER>


4. **Push to DockerHub**  
Authenticates with DockerHub and pushes the image.

5. **Deploy to EC2**  
- Connects via SSH using Jenkins-managed credentials.  
- Pulls the Docker image from DockerHub.  
- Removes the old container (if running).  
- Runs the new container, exposing the app on port `80`.

6. **Post Actions**  
- Success → logs "App deployed successfully".  
- Failure → logs "Build failed, check logs!".  

---

## 🛠️ Better Approach (RPM + S3-based Deployment)

Instead of deploying via Docker images directly, we can use **RPM packaging and S3** for more controlled, production-grade deployments:

### Flow

1. **Build Phase (CI Pipeline)**
- Run `npm install`
- Package the application into a `.tar.gz` archive.
- Create an **RPM package** from the tarball (using `rpmbuild`).
- Upload the RPM to an **S3 bucket** (versioned).

2. **Deployment Phase (CD Pipeline or Manual Trigger)**
- Connect to the EC2 server.
- Pull the required **RPM** from S3 (specific version if needed).
- Install/upgrade the RPM:
  ```bash
  sudo yum install -y myapp-1.0.0.rpm
  OR
  sudo rpm -ivh myapp-1.0.0.rpm
  ```
- Service is restarted automatically via `systemd` (included in RPM scripts).

### Benefits of This Approach
- **Version Control** → Each RPM has a version, allowing easy rollback.
- **No Docker Dependency** → Deployment does not rely on Docker runtime.
- **Lightweight & Faster** → Only the app artifacts are shipped, not the entire Docker image.
- **Immutable Artifacts** → Same RPM is deployed across all environments.
- **Better Ops Integration** → Fits well into existing OS package managers (`yum`, `dnf`).

---

## 📊 Comparison of Approaches

| Feature                | Docker-based Deployment        | RPM + S3 Deployment           |
|-------------------------|--------------------------------|-------------------------------|
| Artifact Storage        | DockerHub                     | S3 Bucket (RPMs)              |
| Rollback                | Pull older Docker image       | Install older RPM             |
| OS-level Integration    | Not native (container only)   | Native (systemd, yum)         |
| Deployment Speed        | Medium (image pull + run)     | Fast (rpm install)            |
| Dependency Packaging    | Inside Docker image           | Inside RPM package            |

---

## ✅ Recommendation

For **quick deployments** or container-native applications → stick with **Docker-based deployment**.  
For **production-grade, version-controlled, and rollback-friendly deployments** → adopt the **RPM + S3 approach**.

---

## 🔮 Next Steps
- Implement a Jenkins job for **RPM packaging**.  
- Create a pipeline stage to **upload RPMs to S3**.  
- Configure deployment jobs to **pull and install RPMs from S3**.  
