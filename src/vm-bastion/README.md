# VM Bastion Learnings

This folder captures reusable, public-safe guidance for Azure Bastion setup and troubleshooting.

## What We Learned

- Use a dedicated subnet named `AzureBastionSubnet`.
- The Bastion subnet must be at least `/26` for Standard SKU scenarios.
- Keep Bastion in the same virtual network as target VMs for the simplest connectivity path.
- `az network bastion ssh` requires Bastion tunneling support.

## Build/Verify Checklist

```bash
# Select subscription
az account set --subscription <subscription-id>

# Verify Bastion exists
az network bastion show -g <resource-group> -n <bastion-name> \
  --query "{name:name,sku:sku.name,provisioningState:provisioningState,enableTunneling:enableTunneling}" -o table
```

If Bastion does not exist:

```bash
az network bastion create \
  -g <resource-group> \
  -n <bastion-name> \
  --vnet-name <vnet-name> \
  --public-ip-address <pip-name> \
  --sku Standard
```

## CLI SSH Requirement

If CLI SSH fails due to tunneling requirements:

```bash
az network bastion update -g <resource-group> -n <bastion-name> --enable-tunneling true
```

Then:

```bash
az network bastion ssh \
  --name <bastion-name> \
  --resource-group <resource-group> \
  --target-resource-id <vm-resource-id> \
  --auth-type password \
  --username <vm-username>
```

## File Transfer via Bastion (SCP)

### Overview

Transfer files between your local machine and an Azure VM through Bastion using SSH tunneling.

### Prerequisites

- Bastion with tunneling enabled (`enableTunneling: true`)
- SSH client installed (Windows 11 includes OpenSSH by default)
- Azure CLI (`az`) in your PATH
- Appropriate file permissions on target VM

### Single-Step File Transfer (Recommended)

**Example: Transfer a file from desktop to VM**

```powershell
# Set variables
$RG = "<resource-group>"
$bastion = "<bastion-name>"
$vm = "<vm-name>"
$vmUser = "<vm-username>"
$localFile = "$env:USERPROFILE\Desktop\<local-filename>"
$targetDir = "/home/<vm-username>/"

# Get VM ID
$vmId = az vm show -g $RG -n $vm --query id -o tsv

# Get VM private IP
$vmPrivateIP = az vm list-ip-addresses -g $RG -n $vm --query "[0].virtualMachine.network.privateIpAddresses[0]" -o tsv

# Transfer file via SCP through Bastion tunnel
scp -o ProxyCommand="az network bastion tunnel --name $bastion --resource-group $RG --target-resource-id $vmId --resource-port 22 --port %p" `
  $localFile "$vmUser@${vmPrivateIP}:$targetDir"
```

### Two-Terminal Tunnel Method (Alternative)

**Terminal 1: Start the Bastion tunnel (keep running)**

```powershell
$RG = "<resource-group>"
$bastion = "<bastion-name>"
$vm = "<vm-name>"

$vmId = az vm show -g $RG -n $vm --query id -o tsv

az network bastion tunnel `
  --name $bastion `
  --resource-group $RG `
  --target-resource-id $vmId `
  --resource-port 22 `
  --port 50000
```

**Terminal 2: Transfer the file**

```powershell
$localFile = "$env:USERPROFILE\Desktop\<local-filename>"
$vmUser = "<vm-username>"
$targetDir = "/tmp/"

# Transfer to VM
scp -P 50000 $localFile "$vmUser@localhost:$targetDir"

# SSH into VM and move to final destination (if needed)
ssh -p 50000 $vmUser@localhost
# Once connected:
sudo mv /tmp/<local-filename> /home/<vm-username>/
sudo chown $vmUser:$vmUser /home/<vm-username>/<local-filename>
exit
```

### Granting Write Access to Target Directories

If the target directory has permission issues:

**On the VM (via SSH):**

```bash
# Grant user read-write access to /home
sudo chown <vm-username>:<vm-username> /home
sudo chmod 755 /home

# Or create a dedicated directory
sudo mkdir -p /home/<vm-username>
sudo chown <vm-username>:<vm-username> /home/<vm-username>
sudo chmod 755 /home/<vm-username>
```

### Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| `Permission denied` | Target directory not writable | Create target dir or use `/tmp` first, then move with sudo |
| `Connection closed` | Tunnel not running | Keep Terminal 1 open, or use ProxyCommand method |
| `No such file or directory` | Local file doesn't exist | Verify file path: `Test-Path $localFile` |
| `iptables` errors | Docker networking issue | Run single-step SCP method, avoid port conflicts |

### Best Practices

1. **Always transfer to `/tmp` first** if target directory has permission constraints
2. **Use `sudo mv`** to move files to restricted directories
3. **Verify file permissions** after transfer: `ls -la /path/to/file`
4. **Keep the tunnel alive** in Terminal 1 when using the two-terminal method
5. **Use the ProxyCommand method** for scripted, automated transfers

## Common Pitfalls

- Correct Bastion exists, but in a different resource group/VNet than expected.
- `enableTunneling` is `false`, which blocks CLI SSH usage.
- Local command context points to the wrong subscription.
- Target directory on VM has restrictive permissions (chmod/chown needed).
- Attempting to transfer to directories without write permissions.
- Not keeping Bastion tunnel alive when using the two-terminal method.

## Public Sharing Notes

- Do not commit real subscription IDs, usernames, passwords, or resource names.
- Use placeholders in scripts and docs.
- Include example commands with `<placeholder>` syntax for clarity.
