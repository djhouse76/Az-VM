# Az-VM

Public-safe Azure VM build templates and operational learnings for Bastion access and Docker host networking.

## Repository Purpose

This repository contains:

- Reusable Azure CLI VM build templates
- Bastion setup and troubleshooting notes
- Docker host networking patterns for controlled ingress and stable egress

All examples are designed to be GitHub-shareable and avoid tenant-specific secrets.

## Repository Structure

- [src/vm-build](src/vm-build)
	- VM creation templates:
		- [src/vm-build/linux-vm.azcli](src/vm-build/linux-vm.azcli)
		- [src/vm-build/windows-vm.azcli](src/vm-build/windows-vm.azcli)
	- Usage notes: [src/vm-build/README.md](src/vm-build/README.md)

- [src/vm-bastion](src/vm-bastion)
	- Bastion build/verify guidance and common pitfalls
	- [src/vm-bastion/README.md](src/vm-bastion/README.md)

- [src/vm-docker-host](src/vm-docker-host)
	- Docker host networking learnings (firewalld-authoritative workflow, ingress allowlists, egress checks)
	- [src/vm-docker-host/README.md](src/vm-docker-host/README.md)

- Historical troubleshooting log:
	- [work/MigVisor-APAC/docker/network-troubleshooting-session-log-2026-08-24.md](work/MigVisor-APAC/docker/network-troubleshooting-session-log-2026-08-24.md)

## Quick Start

1. Pick a VM template from [src/vm-build](src/vm-build).
2. Replace placeholder values (subscription, resource names, usernames, passwords).
3. Run the script in a shell configured with Azure CLI login.
4. Use the Bastion and Docker-host guides for post-build network validation.

## Validation Checklist

### Bastion

- Confirm Bastion is provisioned and in the expected RG/VNet.
- Confirm tunneling configuration matches your CLI SSH usage.
- See [src/vm-bastion/README.md](src/vm-bastion/README.md) for SSH and file transfer guides.

### Docker Host Egress (443)

- Validate host and container HTTPS to `https://management.azure.com`.
- Expected healthy signal can be HTTP `400 MissingApiVersionParameter`.

### Controlled Ingress (80/8900)

- Validate source allowlist behavior for approved source IPs.
- Validate a negative test from a non-allowlisted source IP.

## Security Notes

- Do not commit real passwords, secrets, or tenant-specific credentials.
- Keep scripts template-based with placeholders.
- Review diffs before commit to prevent accidental secret leakage.

## Contributing

Found a bug, have suggestions, or want to improve these templates? 

1. **Fork** this repository to your account
2. Create a **feature branch** for your changes:
   ```bash
   git checkout -b feature/your-improvement
   ```
3. Make your changes and test thoroughly
4. **Push** to your fork:
   ```bash
   git push origin feature/your-improvement
   ```
5. Submit a **Pull Request** back to this repository with:
   - Clear description of the change
   - Reasoning or problem it solves
   - Any testing you've performed

All pull requests are reviewed and greatly appreciated. Please follow the same security practices outlined in the [Security Notes](#security-notes) section.