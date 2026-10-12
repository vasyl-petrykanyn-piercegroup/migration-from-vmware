# ADR-001: VMware Replacement Platform and Selection Gates

- **Status:** Proposed; no production platform selected or approved.
- **Date:** 2026-10-12.
- **Target decision window:** January 2027, subject to the VMware contract timeline.
- **Decision owner:** Project owner; named approver to be confirmed.
- **Scope:** VM replacement platform, on-premises Kubernetes, storage integration,
  availability, and migration readiness. Commercial assessment remains separate.

## Context and Sources

VMware licensing drives this migration. The replacement must reuse the existing
hardware and NetApp storage while adding on-premises Kubernetes capability.
Migration is expected to start no earlier than early 2027. A platform choice
must balance continuity for existing VMs with the desired container capability
and the cost of operating additional infrastructure layers.

This ADR synthesizes the following repository inputs:

- [Transcribed decision brief](platform-decision-brief-input.md), based on the supplied
   September 2026 screenshots and including source
  qualifications and the January 2027 decision rationale.
- [Discussion inputs](platform-discussion-input.md), covering September through
  12 October 2026 and the named platform options.

The original and transcribed briefs are the same underlying source, not
independent evidence. Infrastructure figures below are reported measurements or
estimates from those inputs; this ADR does not independently validate them.
Product capabilities, minimum requirements, and release dates remain source
claims until checked against dated, version-specific vendor documentation.
The brief's missing list of three requested decisions is not reconstructed.

### Reported Estate and Capacity

| Item | Reported baseline | Decision implication |
| --- | --- | --- |
| Estate | Approximately 230 VMs across three environments on 16 blades | Reconcile the inventory before defining migration waves. |
| Production | 12 production blades; estimated consolidation onto 6–7, freeing 5–6 | Validate peak demand and failure reserve before releasing any hosts. |
| Workload storage | All workload data on NetApp; small local disks on blades | Verify external-storage integration and local system-disk requirements. |
| Local capacity | Brief: 224 GB; discussion: 240 GB in RAID 0 | Raw versus usable capacity, units, and disk layout remain unresolved. |
| Intershop retirement | Expected removal of 18 VMs, about one-fifth of production memory and one-quarter of allocated CPU | Confirm retirement and remeasure the remaining estate; allocated CPU is not measured demand. |
| Powered-off VMs | Roughly 20 still consume storage | Decide retention or retirement; do not assume all can be deleted. |
| Storage headroom | Reportedly several times the migration and rollback requirement | Verify physical capacity, growth, snapshots, and temporary migration copies. |
| Locations | Two locations; storage and host distribution are not specified | Site independence and surviving-site capacity remain unproven. |

Do not derive a final migration count by subtracting Intershop and powered-off
VMs from 230: retirement status and overlap are not established.

## Requirements and Decision Drivers

### Constraints Recorded in the Inputs

1. Replace VMware and support the existing VM estate alongside Kubernetes
   workloads. Whether all VMs require lift-and-shift remains unresolved.
2. Reuse existing blades and NetApp storage. No new hardware budget or disk
   replacement approval is available; do not depend on additional dedicated
   control-plane hardware outside the current estate.
3. Use the brief's no-mandatory-platform-licence criterion for the shortlist.
   Support subscriptions and the policy for operating without vendor support
   require a separate, explicit decision.
4. Provide on-premises Kubernetes for development/test and production, including
   an application failover destination for selected fly.io services.
5. Establish a deployment that meets the chosen version's documented hardware
   and storage requirements; do not rely on an unsupported exception to fit.

### Preferences and Unresolved Service Requirements

Prefer reproducible infrastructure as code, manageable recovery and upgrades,
fewer infrastructure layers, familiar operations, and avoiding a second VM
migration. A combined platform is desirable, but the inputs do not establish
that one product must provide every capability.

Before approval, define service-specific recovery time objectives (RTO),
recovery point objectives (RPO), migration downtime, backup retention, and
performance requirements. Also confirm environment isolation, required VM
features, operational ownership, and the scope of fly.io failover. Informal
claims of recovery in minutes are not accepted recovery targets or test results.

## Proposed Decision

Keep production platform selection open until the January 2027 decision window,
provided the contract expiry and required migration runway permit it. Use the
intervening period for inventory, capacity validation, recovery design, and
bounded platform evaluation.

Use **Proxmox VE with Talos Kubernetes clusters hosted in VMs** as the baseline
for evaluation and the preferred fallback, subject to all acceptance gates.
This is an ADR recommendation based on the discussion's operational preferences
and reported team experience, not an approved deployment or a completed proof
of suitability. Validate the actual target versions; the discussion names
Proxmox 9 but does not establish a tested version combination.

Evaluate the proposed **Talos-based combined VM/Kubernetes approach** as a
challenger. First establish the exact product, release, supported architecture,
and VM management capabilities. Do not assume that it is the unnamed product
expected in December 2026 in the brief, or treat that date as a verified release
commitment. Select it only if demonstrated benefits and operational readiness
justify choosing it over the baseline.

Do not adopt Talos with KubeVirt merely as a temporary step toward another
virtualization implementation. It may be evaluated as a durable alternative
if requirements justify it; a planned second migration needs its own explicit
rationale and effort estimate.

### Decision Deadline and Fallback

Propose **15 January 2027** as the evidence cut-off and **31 January 2027** as
the latest selection date. These are new planning proposals, not dates agreed
in the source discussion. Confirm or advance them after checking contract
expiry, migration duration, rollback time, and contingency.

At the cut-off, exclude candidates lacking an available production release or
the required evidence for this estate. Proceed with the Proxmox/Talos baseline
only if it passes the gates below. If it fails, record the failed requirements
and escalate the scope, timing, or budget decision; do not silently waive a
constraint or assume another candidate is ready. Do not extend the schedule
solely to wait for an unverified product release.

## Alternatives and Trade-offs

| Alternative | Reasons to evaluate | Costs, risks, and proposed disposition |
| --- | --- | --- |
| Proxmox VE with Talos tenant clusters | Discussion reports team experience and easier management; provides a concrete VM-first baseline for comparison | Adds hypervisor and guest-cluster recovery stages. Validate NetApp integration, import, IaC, and required Kubernetes integrations. Preferred fallback, not yet qualified. |
| Talos-based combined VM/Kubernetes platform | May reduce layers and avoid a later VM migration, as anticipated in the discussion | Product identity, availability, management functions, storage integration, and recovery are unverified. Time-boxed challenger. |
| Talos with KubeVirt | Named approach for hosting VMs in a Kubernetes-oriented architecture | Discussion anticipates more integration work and possibly another migration. Retain only if evaluated as an intentional long-term design. |
| Harvester | Discussed as an integrated virtualization approach | Local-storage fit and management topology remain unresolved. Hold outside the preferred shortlist pending evidence of a supported fit without hardware purchases. |
| Alternative open-source hypervisor from the brief | Brief describes a mature second option | Product is unnamed and a technical limit is unresolved. Identify it and the limit before meaningful comparison. |
| Commercial enterprise platform from the brief | Brief describes a mature hardware-compatible option | Reported mandatory licence conflicts with the shortlist criterion. Excluded on the input's premise; product and terms are not independently verified. |
| Continue VMware | Preserves the current operating model temporarily | Does not meet the replacement objective. Any contingency extension depends on a separate commercial and timeline decision. |

Named discussion options do not conclusively map to every unnamed category in
the brief. In particular, do not equate Harvester with the originally selected
platform without confirmation. The brief's 500 GB minimum and the discussion's
1 TB Harvester requirement are conflicting source claims, not established
vendor requirements. Tuning replicas or placing workload data on NFS does not
by itself establish compliance with local system-storage requirements.

## Availability and Recovery Architecture

Prefer an independent platform/control plane in each location, with application
and data replication between them, **if required recovery objectives and
measured capacity support that design**. This addresses the brief's reported
failure of a management quorum stretched unevenly across two locations.
It does not establish that the surviving location can serve all workloads.

Validate host distribution, NetApp dependencies, network services, backups,
credentials recovery, and capacity at each location before claiming site
independence. The brief's estimate of roughly two-thirds additional hardware
for two complete systems is an unverified planning estimate. With no hardware
budget, this design is conditional on measured capacity and protected-service
scope.

If two independent systems cannot fit, present a single-system design with
explicitly accepted manual recovery as an alternative. The brief describes
recovery taking hours; the actual duration must be tested and accepted against
service requirements. This is a separate availability choice, not an automatic
consequence of selecting a platform.

Define replication, traffic switching, failback, and write ownership for each
protected application. Test inter-site link failure while both sites remain
running, including fencing or another mechanism that prevents conflicting
writes. Platform availability alone does not deliver fly.io application failover.

For Kubernetes, compare dedicated control-plane capacity within the existing
estate with shared control-plane/workload nodes. Require resource reservations
and contention tests before selecting shared nodes. Document management
dependencies and recovery order, including operation during a Rancher outage
if Rancher is used. Neither Rancher nor external control-plane hardware is an
assumed prerequisite in this ADR.

## Evidence Required for Acceptance

Each finalist must satisfy the applicable gates. Preserve dated vendor sources,
tested versions, configurations, results, and unresolved exceptions with the
evaluation evidence. No finalist currently has a recorded pass in these inputs.

| Gate | Required evidence |
| --- | --- |
| Hardware and capacity | Reconciled post-retirement inventory; local disk layout and usable capacity; representative peak CPU, memory, and storage demand; room for host failure, migration overlap, Kubernetes, and rollback. |
| NetApp and local storage | Supported protocol and integration for the chosen versions; local/system-disk compliance; verified physical array headroom and performance; tested storage-path failure and recovery. NFS is a proposed approach, not a verified current configuration. |
| VM compatibility and migration | Required guest OS and VM features; representative imports; measured downtime and performance; backup restore and rollback, including treatment of writes after cutover. |
| Kubernetes and integrations | Test and production topology; networking, ingress, persistent storage, and isolation; maintained CCM integration if required by selected features, plus the relevant storage integration and version compatibility. |
| IaC and management | Repeatable provisioning and rebuild; required VM lifecycle functions; access controls, operational documentation, and a recovery path when management components are unavailable. |
| Upgrades and recovery | Documented update and rollback or rebuild procedures; measured host, control-plane, management, and storage recovery; evidence resolving the unanswered Talos update question. |
| Site and application continuity | Agreed RTO/RPO; demonstrated capacity for protected services; site-loss and link-partition tests; application replication, traffic switch, failback, and write-conflict prevention. |
| Release and support | Identified product and production release; dated vendor requirements; security maintenance path; a deliberate support policy and assigned operating ownership. |
| Delivery feasibility | Confirmed contract deadline; migration waves, dependencies, rollback window, staffing, and schedule contingency that fit within it. |

## Preparation and Implementation Sequence

1. **October 2026:** Confirm contract dates and decision ownership. Reconcile
   inventory, identify the unnamed candidates, and inspect the proposed
   combined product if available. Record findings without treating a preview
   as production readiness.
2. **Before the decision cut-off:** Agree workload scope and recovery targets;
   confirm Intershop retirement; resolve powered-off VM retention; validate
   storage and peak capacity; evaluate the baseline and challenger against
   the same gates. Complete the separate commercial/support assessment.
3. **January 2027, or earlier if the contract requires:** Refresh capacity
   measurements, compare evidence, select the platform and availability
   topology, and record approval or rejection in this ADR. Assign owners
   and dates to remaining implementation work.
4. **After approval, no earlier than early 2027 under the current plan:**
   Consolidate only after capacity validation; build on released hosts; run
   a representative pilot; migrate in reviewed waves with acceptance and
   rollback criteria. Retire VMware resources only after workload acceptance
   and the agreed rollback-retention period.

This ADR proposes these actions; it does not record them as completed.

## Consequences

The approach preserves a concrete fallback while allowing a combined platform
to demonstrate value before the first production migration. Reusing the
existing estate limits spending but makes storage, capacity, and availability
validation decisive. On-premises production Kubernetes also expands the
operational scope beyond replacing a hypervisor.

The evaluation consumes engineering time and waiting reduces schedule margin.
The contract check and evidence cut-off bound that risk. A Proxmox/Talos choice
introduces additional recovery stages; a combined-platform choice may introduce
less familiar operations. Both require measured recovery and upgrade evidence.
Neither free platform licensing nor an independent control plane proves low
total operating cost or application continuity.

## Approval and Open Decisions

Approval requires an explicit record of the selected product and versions,
support policy, availability topology, accepted RTO/RPO, migration scope,
contract-compatible schedule, and supporting gate results. The current inputs
contain no such approval.

The immediate unresolved decisions are whether the scope is primarily VM
lift-and-shift, which fly.io services need protection, whether two independent
sites fit the existing estate, and whether the proposed January deadlines leave
enough migration time. Revisit this ADR if contract dates, hardware constraints,
service requirements, or evaluation results invalidate the proposed approach.
