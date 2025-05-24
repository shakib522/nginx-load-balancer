# 🚀 DevOps Environment & Deployment Pipeline

---

## 🧩 Problem 1: Environment Setup

### 🔧 Task 1: Manual Setup

#### 🖥️ Virtual Machine Setup

* Create **3 Virtual Machines (VMs)** using a virtualization tool like Oracle VirtualBox or VMware Workstation.
* Install **Ubuntu Server 24.04 (No GUI)** on each VM.
* Assign each VM an IP address from the network **192.168.123.0/24**.
* Install **Docker** inside each VM.

#### 🌐 Reverse Proxy Configuration

* Create a **4th VM** to act as a **reverse proxy**.
* Install **Docker**.
* Install **Nginx** on it (using Docker).

---

### 🤖 Task 2: Automate Setup with Ansible

* Write **Ansible playbooks** to automate the following:

  * Install all system dependencies like **Docker**.
  * Steps to **deploy your web app** inside those VMs.
  * Configure **Nginx** on the 4th VM.

---

## 🌍 Problem 2: Web Application Development

* Develop a **simple web application** to be deployed on the 3 Ubuntu VMs.
* The app should:

  * Display the **hostname** of the VM it is running on.
  * Show the **current commit hash** of the code it was deployed with.
* **Dockerize** the application.

---

## 🔄 Problem 3: Deployment Pipeline

### ⚙️ Use GitHub Actions To:

#### ✅ Continuous Integration (CI)

* **Build** the Docker image.
* **Push** it to **DockerHub**.

#### 🚀 Continuous Delivery (CD)

* **Deploy** the built Docker image inside the **3 VMs** mentioned in Task 1.

---

## 📊 Problem 4: Monitoring

### 🛠️ Monitoring Setup

* Install **Prometheus** and **Grafana**.
* Install Prometheus and Grafana on the **reverse proxy VM**.

### 📡 System Monitoring

* Set up **Prometheus** to monitor the web application running on the 4 Ubuntu VMs.
* Create a **Grafana dashboard** displaying the following metrics:

  * CPU usage
  * Memory usage
  * Disk usage
  * Network statistics

---

## 🔍 Testing

* Test the entire setup by accessing the web application via the domain: **myapp.com**.

---
