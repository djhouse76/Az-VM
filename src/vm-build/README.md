# VM Build Templates

This folder contains public-safe Azure CLI templates for creating Linux and Windows VMs.

## Files

- `linux-vm.azcli`: Linux VM template
- `windows-vm.azcli`: Windows VM template

## Usage Notes

- Replace placeholder values before use (`subscription`, usernames, passwords, VNet names).
- Keep credentials out of source control.
- Validate target VNet/subnet IDs exist before VM create commands.
