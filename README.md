# 🛡️ HA NGINX Reverse Proxy with Keepalived + SSL

This guide walks you through setting up a **Highly Available NGINX Reverse Proxy** using **Keepalived** for VIP failover and **self-signed SSL certificates** to serve traffic securely over HTTPS.

---

## 🔗 Prerequisites

- Ubuntu 22.04 ISO
- VirtualBox, VMware, or any hypervisor
- 4 VMs:
  - `Node-A` and `Node-B` (Backends)
  - `Loadbalancer-1 (LB-1)` and `Loadbalancer-2 (LB-2)` (Reverse Proxies)
- Private IPs configured manually (same subnet)

---

## 🖥️ Backend Node Setup (Node-A and Node-B)

1. Create 2 Ubuntu 22.04 VMs (Node-A and Node-B)
2. Install NGINX on both:
   ```bash
   sudo apt update
   sudo apt install nginx -y
   ```
3. Modify the default index.html page:
   - On Node-A: Replace with "Hello from Node A!"
   - On Node-B: Replace with "Hello from Node B!"
4. Test each node using curl to confirm it serves the expected content.

---

## ⚖️ Load Balancer Setup (LB-1 and LB-2)

1. Create 2 Ubuntu 22.04 VMs (LB-1 and LB-2)
2. Install NGINX on both:
   ```bash
   sudo apt update
   sudo apt install nginx -y
   ```
3. Copy `reverse.conf` from the repository into `/etc/nginx/sites-available/` on both LBs.
4. Enable the config using a symbolic link and restart NGINX:
   ```bash
   sudo ln -s /etc/nginx/sites-available/reverse.conf /etc/nginx/sites-enabled/
   sudo nginx -t && sudo systemctl restart nginx
   ```

---

## 🔁 Keepalived HA Configuration

1. Install Keepalived on both LB-1 and LB-2:
   ```bash
   sudo apt install keepalived -y
   ```
2. Copy `keepalived.conf` from the repository for each LB (`lb-1.keepalived.conf` and `lb-2.keepalived.conf`)
3. Start Keepalived on both:
   ```bash
   sudo systemctl start keepalived
   ```
4. Test VIP failover by accessing `http://<VIP>` and shutting down LB-1 to confirm failover to LB-2.

---

## 🔐 SSL (HTTPS) Support Using Self-Signed Certificates

1. Generate a self-signed SSL certificate on both LB-1 and LB-2:
   ```bash
   sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048    -keyout /etc/ssl/private/nginx-selfsigned.key    -out /etc/ssl/certs/nginx-selfsigned.crt    -subj "/C=IN/ST=Maharashtra/L=Mumbai/O=DevOps/CN=vip.local"
   ```
2. Update the `reverse.conf` to support HTTPS (refer to `lb-1.reverse.conf` and `lb-2.reverse.conf` in repo).
3. Restart NGINX after enabling the updated config.

---

## ✅ Final Architecture Diagram

```
CLIENT
   |
   |--HTTPS--> [ VIP: 192.168.1.100 ]
                   |
          +--------+--------+
          |                 |
     [ LB-1 ]         [ LB-2 ]
          \               /
           \             /
            \           /
           [ Node-A ]  [ Node-B ]
```

---

## 📂 Repository Structure

```
├── lb-1.reverse.conf
├── lb-2.reverse.conf
├── lb-1.keepalived.conf
├── lb-2.keepalived.conf
└── screenshots/
    ├── Keepalived-LB-1-VIP-terminal-output.png
    └── Keepalived-LB-2-VIP-terminal-output.png
```

---

✅ **Made with 💻 by Gaurav Singh**