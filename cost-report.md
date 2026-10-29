# Cost report

Project HIVE ran on an Azure for Students subscription with $100 in credit, so cost control was part of the design from the start. I owned budgeting, monitoring and the teardown plan.

## Spending, October 1 to 15

| Service | Cost |
|---|---|
| Storage | $15.90 |
| Virtual machines | $5.44 |
| Virtual network | $1.77 |
| Bandwidth | under $0.01 |
| Network Watcher | $0.00 |
| **Total** | **$23.11** |

Azure's forecast for the full month was $44.88.

## Burst collection

A Standard_D4s_v3 costs $0.192 an hour, about $140 a month if it runs nonstop. Instead of leaving it on, we ran the honeypot in collection windows and deallocated the VM in between. That kept compute to about 28 hours ($5.44) for the first half of October instead of 360 hours, a saving of roughly $64.

## The budget alert

I set a $19 monthly budget in Azure Cost Management. The alert fired on October 12, when actual cost reached $19.38.

The reason is the most useful thing I learned about cloud cost on this project. Deallocating a VM stops compute billing, but the managed disk, the storage account and the static public IP keep billing whether the VM is on or off. Storage cost almost three times as much as compute, so in a burst-collection design the fixed costs set the floor. A $19 monthly budget was too tight for a full month, even with the VM off most of the time.

## Azure Advisor

| Category | Result |
|---|---|
| Cost | All recommendations followed |
| Security | All recommendations followed |
| Performance | All recommendations followed |
| Reliability | 68% (open items are redundancy a single test VM doesn't need) |
| Operational excellence | 71% |

## Teardown plan

- [ ] Export the dataset, code and dashboards to GitHub
- [ ] Deallocate the VM: `az vm deallocate -g <resource-group> -n <vm-name>`
- [ ] Delete the resource group, which removes the VM, disk, network and storage: `az group delete -n <resource-group>`
- [ ] Confirm in Cost Management that the daily cost reaches $0
