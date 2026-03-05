# SSH Hardening

Documentation of SSH key-based authentication setup, bastion host configuration, and access controls across the lab environment.

## Sections to Cover
- OpenSSH server deployment on DB server and webservers
- ed25519 key pair generation on SOC-Admin bastion host
- Public key distribution to endpoints
- Disabling password authentication and root login
- SSH log forwarding to Splunk
- Firewall rules scoping SSH access to SOC-Admin only
