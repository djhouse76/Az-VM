# VM Build Templates

This folder contains public-safe Azure CLI templates for creating Linux and Windows VMs.

## Files

- `linux-vm.azcli`: Linux VM template
- `windows-vm.azcli`: Windows VM template

## Usage Notes

- Replace placeholder values before use (`subscription`, usernames, passwords, VNet names).
- Keep credentials out of source control.
- Validate target VNet/subnet IDs exist before VM create commands.

## Post-Build Access

Once your VM is created, use Azure Bastion to access it securely:

- **SSH/RDP Access**: See [../vm-bastion/README.md](../vm-bastion/README.md) for Bastion setup and connection guidance.
- **File Transfer**: Transfer files from your desktop to the VM via Bastion — see the "File Transfer via Bastion" section in [../vm-bastion/README.md](../vm-bastion/README.md).

## Common Setup Flow

1. **Create VM** using templates in this folder
2. **Provision Bastion** in the same VNet (see vm-bastion folder)
3. **Access the VM** via Bastion SSH
4. **Transfer files** needed for post-build configuration (applications, configs, scripts)
