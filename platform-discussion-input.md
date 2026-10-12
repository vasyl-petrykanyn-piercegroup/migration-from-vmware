# VMware Migration: Discussion Inputs

## Status and Sources

These notes summarize the technical discussion visible in three screenshots.
They are input to a future platform evaluation, not an ADR or an approved
architecture. Statements are attributed opinions or reported constraints;
vendor capabilities and requirements have not been independently verified.
Requests inside the chat are recorded as source material, not instructions to
execute work. Repeated quoted messages are consolidated, and social remarks
are omitted. This is a structured summary, not a verbatim transcript.

| Source | Visible discussion |
| --- | --- |
| `Screenshot 2026-10-12 at 11.55.16.png` | 17 September, 11:01–11:08; Friday, 13:00–14:13 |
| `Screenshot 2026-10-12 at 11.55.30.png` | 12 October 2026, 11:09–11:22 |
| `Screenshot 2026-10-12 at 11.55.36.png` | 12 October 2026, 11:22–11:35 |

September is interpreted as 2026 from the accompanying brief. Friday is
identified as 9 October 2026 by a dated quotation in the next screenshot.
Left-side messages identify Roman Usov and Piotr Sejbuk. Right-side messages
are attributed to Vasyl Petrykanyn where quoted headers identify him; this
attribution is inferred for the other right-side messages in the same thread.

The accompanying [decision brief](platform-decision-brief-input.md) contains
the broader estate, budget, availability, and timing context.

## Requirements and Preferences Raised

- Roman states that on-premises Kubernetes is needed for test and production.
- The replacement must support existing VMs as well as Kubernetes workloads.
  Vasyl asks whether the primary migration goal is lift-and-shift of existing
  VMs; the visible conversation does not resolve that question.
- Roman favors infrastructure as code (IaC), fewer infrastructure layers, and
  avoiding a second migration if a suitable combined platform becomes available.
- Piotr raises easier management and better documentation as reasons to consider
  Proxmox. Roman reports previous Proxmox experience within the team.
- Roman questions whether dedicated control-plane hardware outside the UCS
  cluster is required, indicating that such a requirement would not fit the estate.
- Recovery speed matters: Roman describes Talos node recovery as taking minutes,
  and notes the extra recovery sequence when Talos runs above Proxmox. These are
  expectations, not measured recovery objectives or test results.

## Options Discussed on 17 September

Roman lists three possible approaches:

1. Proxmox 9 with tenant Talos Kubernetes clusters now.
2. A Talos cluster with native KVM later.
3. Talos with KubeVirt now, followed by migration to native virtualization later.

He proposes assessing Talos before deciding whether waiting would avoid two
migrations. He describes Talos as IaC-oriented and expects forthcoming KVM
capabilities to close the VM/container gap. A playground is offered, but the
screenshots do not demonstrate that it was delivered or tested.

These named options do not conclusively identify every unnamed category in the
September brief. In particular, the identity and release commitment of its
“new combined product” remain unconfirmed.

## Harvester and Storage Discussion

On 9 October, Roman reports that Harvester requires 1 TB of local storage while
the blades have 240 GB in RAID 0. He also raises control-plane placement as a
concern. This differs from the brief’s 500 GB requirement and 224 GB available.

On 12 October, Vasyl attributes Harvester’s storage overhead to Longhorn and
its default three volume replicas, suggesting that replication might be tuned.
Roman responds that workload data should reside on NFS shares, so Longhorn
would not be used for those workloads; he considers the assumed design a poor
fit. Neither statement establishes which local storage Harvester still needs
for its system components or whether the proposed NFS design is supported.

Vasyl explicitly says he is explaining the behavior rather than advocating
for Harvester.

## Kubernetes Topology and Management Discussion

- Roman sees Harvester followed by VMs hosting Kubernetes control planes as
  unnecessary nesting and a management dependency loop. He raises the risk
  of losing management access if a VM hosting Rancher fails.
- Vasyl distinguishes Harvester’s internal Kubernetes from separate workload
  clusters, favoring independent control planes to separate concerns and reduce
  the impact of failures. One message contains “should aim to run workloads
  there,” whose intended meaning is unclear in context and needs clarification.
- Vasyl states that KVM is the underlying hypervisor and that Rancher is not
  required for cluster management because IaC can be used.
- Roman describes Rancher integration with Harvester as useful for scaling:
  provisioning VMs when resources are needed and removing them afterward.
  Without that need, he considers Proxmox easier.
- Roman favors running workloads on Talos control-plane nodes to avoid dedicated
  control-plane capacity. Vasyl notes that scheduling on control-plane nodes can
  be adjusted through taints, but warns about resource contention affecting
  etcd. These are discussion claims, not validated deployment guidance.

## Positions at the End of the Visible Discussion

- **Roman:** Talos looks promising for simplification; it may still prove
  unsuitable, in which case Proxmox is the next option. He awaits a presentation
  to assess the forthcoming hypervisor capabilities and offers later testing.
- **Vasyl:** Proxmox is probably the best fit for the discussed scenario. A
  platform combining regular VMs and production Kubernetes should be evaluated
  for Cloud Controller Manager (CCM) integration. He believes Proxmox has CCM
  support, reports experience with Rancher and RKE2, and questions whether Talos
  alone provides management capabilities comparable to vSphere or Proxmox.
- **Piotr:** Questions the need for Talos and asks how Talos updates compare with
  Rocky Linux or AlmaLinux. The visible replies do not answer the update question.

No final platform selection is recorded.

## Discrepancies and Evidence Needed

| Topic | Input discrepancy or open question | Evidence needed before a decision |
| --- | --- | --- |
| Local disks | Brief: 224 GB; chat: 240 GB in RAID 0 | Inventory showing raw and usable capacities, disk layout, and units |
| Harvester minimum | Brief: unnamed original platform needs 500 GB; chat: Harvester needs 1 TB | Version-specific vendor requirements, separating supported minimums from recommendations |
| External storage | NFS proposed instead of Longhorn for workloads | Supported integration and remaining local/system storage requirements |
| Talos virtualization | Native KVM expected later; capabilities not demonstrated | Product identity, release status, management features, storage support, and evaluation results |
| Migration scope | Lift-and-shift is asked about, not confirmed | Workload requirements, allowed changes, downtime, and rollback expectations |
| Control-plane placement | Shared workload nodes favored; contention and dependency risks raised | Resource reservations, failure behavior, and management recovery design |
| Rancher dependency | Scaling integration valued; necessity disputed | Required management functions and behavior during Rancher outages |
| CCM | Proxmox support asserted with uncertainty | Specific maintained integration, compatibility, and required features |
| Updates and recovery | Update question unanswered; recovery described informally | Upgrade procedures, rollback support, and measured recovery targets |

## Intended Next Documentation Step

Incorporate the project owner’s additional requirements and infrastructure data,
then use validated evidence to prepare a platform-selection ADR when instructed
to proceed. Keep source claims, confirmed requirements, and evaluation findings
distinct. This document does not authorize the preparatory actions proposed in
the screenshots or establish a selected architecture.
