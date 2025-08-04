# Private DNS Ad-Blocking and Remote Access with Pi-hole + Unbound + Tailscale

## 👋 Why I Built This

I care about online privacy and wanted to block ads and trackers across all my devices, not just in browsers. Public DNS providers like Google or Cloudflare are fast, but often log user data. I wanted full control over DNS resolution and to keep it private.

My internet provider (Jio Fiber) uses CGNAT, so I don’t get a public IP address. That makes remote access tricky. This project uses Tailscale to solve that problem while offering secure DNS resolution and ad-blocking.

---

## 🔍 Problems Solved

- Block ads and trackers across all devices
    
- Encrypt and control DNS traffic (no third-party logging)
    
- Secure, recursive DNS resolution via Unbound
    
- Enable remote DNS and Pi-hole admin access even behind CGNAT
    
- Learn real-world Linux, Docker, VPN, DNS, and firewall management
    
![Architecture](/screenshots/Architecure-Pihole+Docker.png)

---

## 🧰 Tools Used

- **macOS** (host)
    
- **Colima** to run Docker with Lima backend on macOS
    
- **Docker** for containerization
    
- **Pi-hole** (ad-blocking DNS)
    
- **Unbound** (recursive DNS resolver)
    
- **Tailscale** (zero-config WireGuard-based VPN)
    

---

## 🛠️ How I Built It

### 1. Setup Docker + Colima

Install Colima and start it:


brew install colima
colima start --memory 2 --cpu 2 --disk 15


Ensure Docker is pointed to Colima:


export DOCKER_HOST=unix://$HOME/.colima/default/docker.sock


### 2. Create Docker Compose Setup

Created `docker-compose.yml` with services:

- **Unbound** listening on port `5335`
    
- **Pi-hole** using Unbound as upstream DNS
    

### 3. Generate Tailscale Auth Key

- Went to Tailscale admin panel
    
- Created an **ACL tag** (e.g. `tag:pihole`)
    
- Generated an **auth key** tagged with `tag:pihole`
    
- Used that auth key in Docker Compose for Tailscale container
    

### 4. Set ACL Tag Permissions

```json
"tagOwners": {
  "tag:pihole": ["user:your_email@example.com"]
},
"acls": [
  {
    "action": "accept",
    "src": ["tag:pihole"],
    "dst": ["*:*"]
  }
]
```

### 5. Start Services


docker compose up -d


Tailscale container auto-joined the network, Pi-hole was accessible locally.

![pihole-dashboard](/screenshots/Pihole-admin-dashboard.png)

### 6. Add Tailscale Global DNS

- Opened [Tailscale DNS settings](https://login.tailscale.com/admin/dns)
    
- Added **Pi-hole container’s Tailscale IP** as a global nameserver (port `53`)
    
- Disabled `MagicDNS`

![Tailscale-DNS](/screenshots/Tailscale-dns.png)
    

### 7. Remote Access

Tested DNS queries from remote Tailscale-connected devices:


nslookup google.com 100.x.x.x
# Expected output:
# Name: google.com
# Address: 142.251.221.110

![remote-access](/screenshots/nslookup.png)


## 🤯 What I Learned

- Difference between recursive and forwarded DNS
    
- Tailscale ACLs and DNS overrides
    
- Docker networking and port mappings
    
- Unbound configuration for Pi-hole
    
- Colima/Lima integration for macOS-based Docker usage


![unbound](/screenshots/unbound-logs.png)
    

---

## 🚀 Future Plans

- Add DNS-over-HTTPS fallback using Cloudflare
    
- Use Raspberry Pi or always-on NUC
    
- Integrate 2FA for Pi-hole admin
    
- Add logging and monitoring for DNS performance
    

---

## ✨ License

This project is licensed under the [MIT License](LICENSE).
You're free to use, modify, and share it — personally or commercially.

Feel free to fork it, improve it for your own setup, or share with others!

---