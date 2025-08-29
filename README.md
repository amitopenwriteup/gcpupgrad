
# ☁️ Google Cloud Platform (GCP) Labs & Projects

This repository contains a **collection of hands-on labs, project files, and sample applications** for learning and practicing with **Google Cloud Platform (GCP)** services. It includes end-to-end scenarios covering compute, storage, networking, serverless, containerization, CI/CD, monitoring, and real-world applications (e.g., **BookMyShow clone**).

---

## 📂 Repository Contents

### 🔹 GCP Core Services

* `Compute instance.docx` – Creating and managing VM instances
* `GCS.docx` – Working with Google Cloud Storage
* `Cloud sql.docx` – Configuring Cloud SQL databases
* `VPC Network.docx`, `gcp-vpc.docx` – Setting up networking in GCP
* `Instance group for webserver.docx`, `Instance group gcp.docx` – Instance groups and load balancing
* `Lifecycle.docx` – Instance lifecycle management

### 🔹 Kubernetes & Containers

* `GKE.docx` – Running workloads on GKE (Google Kubernetes Engine)
* `gkecloudbuild.docx` – Integrating GKE with Cloud Build
* `Cloud Run Container.docx`, `GCP cloud run function.docx` – Deploying containers on Cloud Run

### 🔹 Serverless & Functions

* `Google Cloud FunctionsLab.docx` – Using GCP Functions
* `App Engine lab.docx` – Deploying apps on App Engine

### 🔹 Security & IAM

* `Service Account.docx`, `Create Service Account.docx`, `SA-JSON file.docx` – IAM & service accounts
* `Create Alert policy.docx` – Monitoring & alerting

### 🔹 Monitoring & Observability

* `Dashboards.docx` – Cloud Monitoring dashboards
* `gcp outputblock.docx`, `instanceblockgcp.docx` – Output & instance-level monitoring

### 🔹 Real-World Projects

* `Bookmyshow.docx`, `PHP bookmyshow.docx` – BookMyShow clone using PHP & GCP
* `bookmyshowgcpproject` – Project folder with PHP files (`add.php`, `edit.php`, `delete.php`, `db_connect.php`, `index.php`)
* `index.html` – Sample frontend

### 🔹 Masterclass & Capstone Projects

* `Master class Proj 1.docx`, `Proj 2.docx`, `Proj 3.docx` – Structured GCP projects
* `MasterClass Project.docx`, `transacation Masterclass.docx` – Advanced use cases
* `CAPSTONE Project (2).docx`, `GCP Project2025 (1).docx` – Full project implementation

### 🔹 CI/CD & DevOps

* `CICD.docx`, `CICD.pptx` – Setting up CI/CD pipelines on GCP
* `Artifact Registry.docx` – Using Artifact Registry for images/packages

---

## 🛠️ How to Use

1. Clone the repository:

   ```bash
   git clone https://github.com/<your-org>/<your-repo>.git
   cd <your-repo>
   ```

2. Explore labs by opening the `.docx` and `.pptx` files.

3. For the BookMyShow project:

   * Deploy the PHP app (`index.php`, `add.php`, etc.) on **App Engine** or **Compute Engine**.
   * Configure database using **Cloud SQL**.
   * Connect via `db_connect.php`.

4. Use `gkecloudbuild.docx` and `CICD.docx` to build CI/CD pipelines for containerized deployments.

---

## 🎯 Learning Outcomes

By completing the labs and projects, you will gain hands-on experience in:

* Deploying apps on **Compute Engine, App Engine, Cloud Run, and GKE**
* Setting up **VPC, subnets, and firewalls**
* Managing **IAM & service accounts** for secure access
* Implementing **CI/CD pipelines** using Cloud Build & Artifact Registry
* Monitoring workloads with **Cloud Monitoring & Dashboards**
* Building real-world projects like **BookMyShow clone** on GCP
