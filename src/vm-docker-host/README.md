# VM Docker Host Networking Learnings

This folder documents practical, public-safe learnings for container egress and controlled ingress on an Azure Linux VM running Docker.

## Key Findings

- Keep one firewall authority on host systems using firewalld/nft backend.
- Avoid mixing ad-hoc direct nft manipulation with firewalld-managed policy.
- Validate both runtime and permanent firewalld state.
- For Azure Management API reachability checks, expected success can be HTTP `400 MissingApiVersionParameter`.

## Verified Egress Pattern (Container -> Azure Management)

Allow Docker bridge egress with explicit forward/NAT rules:

```bash
sudo firewall-cmd --zone=trusted --add-source=172.18.0.0/16
sudo firewall-cmd --zone=public --add-masquerade
sudo firewall-cmd --direct --add-rule ipv4 filter FORWARD 0 -i <docker-bridge> -o eth0 -s 172.18.0.0/16 -p tcp --dport 443 -j ACCEPT
sudo firewall-cmd --direct --add-rule ipv4 filter FORWARD 0 -i eth0 -o <docker-bridge> -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo firewall-cmd --direct --add-rule ipv4 nat POSTROUTING 0 -s 172.18.0.0/16 -o eth0 -j MASQUERADE
```

Validate:

```bash
curl -4 -I -m 10 https://management.azure.com
sudo docker exec <container-name> sh -c "curl -4 -I -m 10 https://management.azure.com"
```

Expected healthy response class:

- Host/container can resolve and connect over TLS.
- Response can be `HTTP/1.1 400` with `MissingApiVersionParameter`.

## Controlled Ingress Pattern (Allowlist Source IPs)

When ingress should be limited to approved source hosts:

```bash
sudo firewall-cmd --zone=public --add-rich-rule='rule family=ipv4 source address=<source-ip-1> port protocol=tcp port=80 accept'
sudo firewall-cmd --zone=public --add-rich-rule='rule family=ipv4 source address=<source-ip-1> port protocol=tcp port=8900 accept'
```

Repeat for each approved source IP and persist:

```bash
sudo firewall-cmd --runtime-to-permanent
```

## Troubleshooting Sequence

1. Confirm listeners:
```bash
ss -lntp | egrep ':(80|443|8900) '
```

2. Confirm source-to-target test:
```powershell
Test-NetConnection <target-ip> -Port 80
```

3. Use packet capture to identify drop point:
```bash
sudo tcpdump -ni eth0 'host <source-ip> and tcp and (port 80 or port 8900)'
```

Interpretation:

- SYN on `eth0` with no success from client: issue is likely host policy/return path handling.
- No SYN on `eth0`: investigate upstream Azure network policy.

## Operational Guardrails

- Export state before changes.
- Apply runtime fixes, validate, then persist.
- Re-check runtime/permanent alignment after reload.
- Never commit secrets to repo (passwords, tenant secrets, real credentials).
