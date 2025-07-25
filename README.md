# NGINX High Availability Load Balancer with Keepalived

This guide walks you through the steps to create a highly available reverse proxy setup using NGINX and Keepalived on Ubuntu 22.04 VMs.

---

## 🔗 Prerequisites

* Download Ubuntu 22.04 ISO: [https://releases.ubuntu.com/22.04/](https://releases.ubuntu.com/22.04/)
* VirtualBox, VMware, or any hypervisor to run VMs

---

## 🖥️ Backend Node Setup (Node-A and Node-B)

### 1. VM Configuration

* **VM Name**: `Node-A` and `Node-B`
* **CPU**: 1 Core
* **RAM**: 2 GB

### 2. Installation & Setup (Run on both VMs)

```bash
sudo apt update
sudo apt install nginx -y
sudo service ssh start
```

### 3. Modify HTML Page

#### On Node-A

```bash
echo "<h1>Hello from Node A!</h1>" | sudo tee /var/www/html/index.html
```

#### On Node-B

```bash
echo "<h1>Hello from Node B!</h1>" | sudo tee /var/www/html/index.html
```

### 4. Test Output

```bash
curl http://<Node-A-IP>  # Should return "Hello from Node A!"
curl http://<Node-B-IP>  # Should return "Hello from Node B!"
```

---

## ⚖️ Load Balancer Setup (LB-1 and LB-2)

### 1. VM Configuration

* **VM Name**: `Loadbalancer-1` and `Loadbalancer-2`
* **CPU**: 1 Core
* **RAM**: 2 GB

### 2. Install NGINX

```bash
sudo apt update
sudo apt install nginx -y
sudo service ssh start
```

### 3. Configure Reverse Proxy

#### On both LB VMs:

```bash
sudo nano /etc/nginx/sites-available/reverse.conf
```

Paste the content from the repo files:

* `lb-1.reverse.conf`
* `lb-2.reverse.conf`

Enable the config:

```bash
sudo ln -s /etc/nginx/sites-available/reverse.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl restart nginx
```

### 4. Test Output

```bash
curl http://192.168.1.41
curl http://192.168.1.42
```

Expected output (round-robin):

```
Hello from Node A!
Hello from Node B!
```

---

## 🔁 Keepalived HA Configuration

### 1. Install Keepalived

```bash
sudo apt update
sudo apt install keepalived -y
```

### 2. Create Config File

```bash
sudo nano /etc/keepalived/keepalived.conf
```

Paste the contents from:

* `lb-1.keepalived.conf`
* `lb-2.keepalived.conf`

### 3. Start the Service

```bash
sudo service keepalived start
```

### 4. Test VIP Failover

* Access the VIP from browser or terminal:

```bash
curl http://192.168.1.100
```

* Simulate failure by stopping keepalived or powering off `LB-1`
* `LB-2` should take over the VIP automatically

### 5. Outputs

Refer to the `screenshots` folder:

* `Keepalived-LB-1-VIP-terminal-output.png`
* `Keepalived-LB-2-VIP-terminal-output.png`

---

## ✅ Conclusion

This setup ensures:

* Round-robin load balancing between backend nodes
* Automatic failover using Keepalived
* A robust and fault-tolerant web service architecture

---

## 📂 Repository Structure

```
├── lb-1.reverse.conf
├── lb-2.reverse.conf
├── lb-1.keepalived.conf
├── lb-2.keepalived.conf
└── screenshots
    ├── Keepalived-LB-1-VIP-terminal-output.png
    └── Keepalived-LB-2-VIP-terminal-output.png
```

---

Made with ☕ and 💻 by Gaurav
