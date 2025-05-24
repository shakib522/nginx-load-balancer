


# 🐳 Load Balancing with Docker and Nginx on Multiple Virtual Machines

This project sets up **4 Ubuntu Server VMs** using VirtualBox:

* **3 VMs as backend servers running Nginx**
* **1 VM as a Load Balancer using Nginx and Docker**

---

## 📦 Requirements

* [Oracle VirtualBox](https://www.virtualbox.org/)
* Ubuntu Server 24.04 ISO images [download](https://ubuntu.com/download/server)
* Internet access on host machine
* Basic Linux command line knowledge

---

## 🖥️ VM Network Configuration

### 1. Configure Network Adapters in VirtualBox

Each VM should have **2 adapters**:

* **Adapter 1**: NAT (for internet)
* **Adapter 2**: Host-Only Adapter (for internal VM-to-VM and Host-to-VM communication)

Create a **Host-Only Network (vboxnet0)**:

1. Go to **VirtualBox → Tools → Network → Host-Only Networks**
2. Click **Create**, then edit:

   * **IPv4 Address**: `192.168.123.1`
   * **Mask**: `255.255.255.0`
3. Disable **DHCP Server**

---

### 2. Configure Static IP on Each VM

Edit file: `/etc/netplan/50-cloud-init.yaml`

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.123.X/24  # Change X uniquely per VM
      nameservers:
        addresses: [8.8.8.8]
```

Run:

```bash
sudo netplan apply
```

Assign these IPs:

* VM1: `192.168.123.4`
* VM2: `192.168.123.2`
* VM3: `192.168.123.3`
* Load Balancer VM: `192.168.123.5`

---

## 🔒 SSH into VMs

Install OpenSSH server on all VMs:

```bash
sudo apt install openssh-server -y
```

From host, SSH into VM:

```bash
ssh sakib@192.168.123.X
```

---

## 🌍 Setup Nginx Backend Servers on VM1, VM2, VM3)


---

Each of these VMs will host a simple Nginx server using Docker and display a unique message to help verify load balancing.

---

### 🚀 Run Nginx Container

On **each VM (192.168.123.2, 192.168.123.3, 192.168.123.4)**, run:

```bash
sudo docker run -d --name webserver -p 80:80 nginx
```

---

### ✏️ Customize the Web Page

1. Enter the running container:

   ```bash
   sudo docker exec -it webserver bash
   ```

2. Install `nano` inside the container:

   ```bash
   apt update && apt install -y nano
   ```

3. Navigate to the default web root:

   ```bash
   cd /usr/share/nginx/html
   ```

4. Edit the main page:

   ```bash
   nano index.html
   ```

5. Replace the content with this line (change the VM number accordingly):

   ```html
   <h1>Welcome to nginx from VM [2|3|4]!</h1>
   ```

   Example for VM2:

   ```html
   <h1>Welcome to nginx from VM 2!</h1>
   ```

6. Save and exit:

   * Press `CTRL + X`
   * Press `Y` to confirm
   * Press `ENTER` to save

---

### ✅ Test the Nginx Servers

From your browser, hit following url for verify each Nginx server :

```bash
 http://192.168.123.2/
 http://192.168.123.3/
 http://192.168.123.4/
```

You should see a unique welcome message in your browser for each VM like:

```
  Welcome to nginx from VM 2!
```

Repeat this for VM3 and VM4 to ensure they're serving the correct content.



---

## 🌐 Setup Load Balancer on VM4 (192.168.123.5)

### 1. Create working directory

```bash
mkdir ~/reverse-proxy
cd ~/reverse-proxy
```

### 2. Create `nginx.conf`

```nginx
worker_processes 1;

events {
  worker_connections 1024;
}

http {
  upstream backend_servers {
    server 192.168.123.2;
    server 192.168.123.3;
    server 192.168.123.4;
  }

  server {
    listen 80;

    location / {
      proxy_pass http://backend_servers;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
    }
  }
}
```

### 3. Run Load Balancer Container

```bash
docker run -d \
  --name reverse-proxy-round-robin \
  -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx
```

---

## 🔁 How Round Robin Works

**Round Robin** load balancing rotates incoming HTTP requests **evenly** across backend servers (VM1, VM2, VM3).

For example:

```
Request 1 → VM1
Request 2 → VM2
Request 3 → VM3
Request 4 → VM1
```

---

## ✅ Testing

From your browser hit :

```bash
  http://192.168.123.5
```

Refresh multiple times in browser:

```bash
 http://192.168.123.5
```

You should see rotating responses from different VMs.

---

## 🧪 Debugging Tips

* Ensure all VMs are running and accessible via `ping 192.168.123.X`
* Inside each VM, run: `curl localhost` to test nginx container
* From load balancer VM, test each backend directly: `curl http://192.168.123.2`
* View logs: `docker logs reverse-proxy-round-robin`
* Restart Nginx container if config changed: `docker restart reverse-proxy-round-robin`

---

