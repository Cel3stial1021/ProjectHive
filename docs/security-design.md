# Security design

This page covers how Project HIVE was secured. A honeypot is a strange thing to protect: the decoys have to be exposed on purpose, so the work is about making sure an attacker only ever reaches the decoys, and that nothing they do there can spread, erase our evidence, or run up the bill. I owned the Azure side of this, and the table notes where a teammate owned a control.

## Threats and controls

| Threat | Control | Owner |
|---|---|---|
| An attacker breaks out of a decoy and uses our VM to attack other systems | Outbound traffic restricted in the NSG, then re-tested in Sprint 2 to confirm it held | Carlos |
| The management side of the server is reachable from the internet | T-Pot management ports restricted in the NSG; SSH on a non-standard port with key authentication and individual accounts | Carlos (NSG), Nigel (SSH and accounts) |
| Logs are lost or altered if the VM is compromised or deleted | Logs copied to a separate storage account with blob versioning and 30-day soft delete | Carlos |
| Stolen storage keys or anonymous reads expose the logs | Shared key access disabled, so every request needs Microsoft Entra ID authorization; anonymous blob access disabled | Carlos |
| Logs are intercepted in transit | Secure transfer required, TLS 1.2 minimum | Carlos |
| The storage endpoint is open to the whole internet | Public network access limited to selected networks and IP addresses | Carlos |
| Our own logins pollute the attack data | The export pipeline drops traffic to T-Pot's management ports and private addresses | Nigel |
| Cost runs away on a student credit | Monthly budget with an alert, VM deallocated between collection windows | Carlos |
| Resources keep running and billing after the project | Written teardown plan | Carlos |

## Network security group

The NSG is the main control in the whole project. Inbound ports 1 to 64000 are open so attackers can reach every decoy. T-Pot's management ports sit above that range and are restricted, so the admin side of the server is never part of the attack surface. Outbound traffic is restricted because the biggest risk with a honeypot is not someone getting in (that is the point), it is someone getting in and then using the VM to attack others.

## Log storage

| Setting | Value |
|---|---|
| Account type | StorageV2, Standard, locally redundant, Hot tier |
| Shared key (account key) access | Disabled |
| Anonymous blob access | Disabled |
| Secure transfer | Required, TLS 1.2 minimum |
| Blob versioning | Enabled |
| Blob and container soft delete | Enabled, 30 days |
| Public network access | Selected virtual networks and IP addresses only |
| Tags | project: honeypot |

## Reviews

In Sprint 3 I wrote a security review and incident runbook, reviewed the threat model from our design phase, and audited the NSG rules. Azure Advisor showed every security and cost recommendation already followed.

## Known gaps

- Everything runs on one VM, so there is no redundancy. Advisor's reliability score (68%) reflects that, and it was an accepted trade-off for a test honeypot.
- Network Watcher is enabled, but VNet flow logs were not turned on yet. That would be the next step for network-level visibility.
- The management ports live on the same public IP as the decoys. They are restricted by source in the NSG, but a separate management path (for example Azure Bastion) would remove them from the public IP entirely.
