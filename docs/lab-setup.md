# Lab setup and operating notes

## Observed environment

| Component | Configuration |
|---|---|
| Host | Windows 11 Pro, Intel i7-12700F, 16 GB RAM, NVMe SSD |
| Hypervisor | Oracle VirtualBox 7.2.4 |
| SIEM VM | Ubuntu Server 24.04.3 LTS, 8 GB RAM, 4 vCPUs, approximately 80 GB virtual disk |
| SIEM components | Wazuh manager, indexer, dashboard, and Filebeat; Wazuh 4.14.9 |
| Endpoint | Existing Windows 10 Home VM; Wazuh agent 4.14.9 |
| Agent identity | SOC-WIN10, agent ID 001 |
| SIEM lab IP | 192.168.56.101 |
| Endpoint lab IP | 192.168.56.102 |

Reducing the Windows VM allocation to 4 GB was recommended during setup; its final allocation was not independently captured. RAM on the host is constrained when both VMs run. A guest CPU soft-lockup warning was observed once; its cause was not established.

## Network

Each VM was configured with NAT plus the same VirtualBox host-only network.

- NAT provides outbound connectivity for downloads.
- Host-only networking connects the host and the two guests.
- Agent telemetry was tested against TCP 1514 on the manager.
- Dashboard access uses HTTPS on the manager's host-only address.
- An alternative dashboard access path forwards host `127.0.0.1:8443` to guest TCP 443.
- SSH was enabled on Ubuntu and OpenSSH Server was installed and started on the Windows guest to allow copying commands from the host.

The host-only adapter is not a fully isolated malware sandbox: guests can communicate with the host, and NAT permits outbound access. This project used controlled authentication attempts, not live malware.

Host-only addresses were assigned using DHCP. They can change; recheck addresses after network changes before reusing these examples.

## Setup summary

1. Installed Ubuntu Server, applied operating-system updates, and rebooted.
2. Installed Wazuh central components using the official 4.14 installation assistant.
3. Enabled Ubuntu's second adapter using a separate Netplan configuration.
4. Attached the Windows guest to the same host-only network.
5. Installed the agent with manager address 192.168.56.101 and agent name SOC-WIN10.
6. Verified the agent became Active.
7. Checked the endpoint's Logon audit policy: Success and Failure.
8. Validated individual and correlated authentication-failure alerts.

## Troubleshooting findings

### Dashboard API timeouts

The dashboard intermittently reported a 20-second API timeout. Checks showed services active, successful API responses in logs, and an unauthenticated API response from localhost, including under the dashboard service account. The page later became usable after a reboot. A definitive root cause was not established.

### Alerts hidden by the time range

The manager's alerts.json contained endpoint alerts, Filebeat's output test passed, and index queries confirmed endpoint documents existed. An absolute time range exposed 428 endpoint alerts while a relative range had shown none.

Clock comparisons showed discrepancies between the host and guests. After host time synchronization, alerts appeared under Last 24 hours. The Windows guest clock was also synchronized before authentication testing. These observations support a time-range/clock issue; they do not establish that clock skew caused the separate API timeouts.

### Old Filebeat errors

Rotated logs contained connection failures and indexing timeouts dated 30 October during setup. Subsequent index queries and successful detection screenshots confirmed indexing was working during the experiment. The old errors were not treated as proof of a current outage.

## Before reusing the lab

- Start both VMs and verify the agent is Active.
- Verify clocks and time synchronization on the host and guests.
- Confirm current lab IP addresses.
- Use dedicated test identities.
- Keep credentials and private keys out of screenshots and Git.
- Review SSH startup settings if access is needed after reboot; automatic startup was not verified during this session.
