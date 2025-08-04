#### ❌ Problem: Unbound not responding on port 5335

- **Cause**: Colima networking vs host port conflict
    
- **Fix**: Verified Unbound container was listening on `172.22.0.10:5335`
    
    - Used `ss -uln` inside the container
        

#### ❌ Problem: Remote devices could not resolve DNS

- **Cause**: Pi-hole wasn’t properly using Unbound as upstream DNS
    
- **Fix**: Corrected `PIHOLE_DNS_` environment variable in `docker-compose.yml`
    

#### ❌ Problem: Couldn’t reach Pi-hole DNS remotely

- **Cause**: Tailscale wasn't advertising DNS port
    
- **Fix**: Added firewall rules inside the container using iptables
    

    iptables -A INPUT -s 100.0.0.0/8 -p udp --dport 53 -j ACCEPT
    iptables -A INPUT -s 100.0.0.0/8 -p tcp --dport 53 -j ACCEPT
    
    

#### ❌ Problem: Docker daemon not responding

- **Cause**: Docker socket not found after restart
    
- **Fix**: Re-exported Docker socket after `colima start`:
    
    
    export DOCKER_HOST=unix://$HOME/.colima/default/docker.sock

    

#### ❌ Problem: socat port forwarding failed (port already in use)

- **Fix**: Identified `limactl` occupying UDP 5335 using `lsof -iUDP:5335`, then killed and restarted socat
    