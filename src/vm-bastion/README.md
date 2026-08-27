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

## Common Pitfalls

- Correct Bastion exists, but in a different resource group/VNet than expected.
- `enableTunneling` is `false`, which blocks CLI SSH usage.
- Local command context points to the wrong subscription.

## Public Sharing Notes

- Do not commit real subscription IDs, usernames, or passwords.
- Use placeholders in scripts and docs.
