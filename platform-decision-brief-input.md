# Leaving VMware: Platform Decision Brief

> Source input, not an approved decision or ADR. Product requirements, capacity estimates, and release dates below are claims from the September 2026 brief; they have not been independently verified.

## Source and Scope

- Author shown in the chat: Roman Usov; shared on 17 September at 10:57 (year inferred as 2026 from the brief).
- Screenshots: `Screenshot 2026-10-12 at 11.54.55.png` and `Screenshot 2026-10-12 at 11.55.06.png`.
- Transcription is based on the supplied screenshots; no separate source text is retained in this repository.
- Markdown formatting is editorial; the brief’s wording and unnamed platform categories are preserved.
- The brief refers to three decisions listed at the end, but no such list is visible in the supplied screenshots. It is not reconstructed here.

**Prepared September 2026. For decision in January 2027.**

## The ask

We are replacing VMware. This brief sets out what we have found, what the options are, and why the
platform choice should be made in January rather than now. Three decisions are needed, listed at the
end.

Commercial and licensing detail is handled separately and is not covered here.

## Why we are replacing it

VMware licensing is the driver. The replacement therefore has to avoid carrying a large licence of
its own, which rules out some otherwise capable products and is a constraint running through
everything below.

## What we run today

Around 230 virtual machines across three environments, on 16 blade servers, with all data on the
NetApp storage array. The blades have small internal disks and hold no workload data; everything
lives on the array. There is no budget for new hardware, so the replacement has to run on what we
already own.

Separately, our container workloads run at Amazon and at fly.io, because we have no equivalent
capability in our own data centre. Part of this project is to build that capability, both for
development and test use and so that we have somewhere to fail over to if fly.io has an outage.

## What we have established since the original plan

The platform originally chosen does not fit our hardware. It requires 500 GB of internal disk per
server for production use. Our blades have 224 GB. Replacing the disks across the estate was not
approved, and running below the vendor's stated minimum means running a configuration the vendor does
not support. This is the single largest finding and it is what reopened the platform question.

We can free enough servers to build the new platform without buying anything. Measured from
current inventory, the production workload can be consolidated onto six or seven of the twelve
production blades, leaving five or six free to build the replacement. Storage has several times the
free space needed for the migration and for keeping rollback copies. Both figures need re-measuring
closer to the start, but the headroom is comfortable rather than marginal.

The Intershop move shrinks the job before we start. Retiring the current web, application and
database servers for Intershop removes 18 machines, which is a fifth of production memory and a
quarter of production processor allocation. Planning against what remains rather than against
today's estate materially reduces the work.

Every serious candidate is free to run. The realistic options are all open-source and carry no
mandatory licence. Support is a separate decision we can take later, on its merits, rather than a
precondition for starting. One candidate has no free path and is out for that reason.

## Why the decision belongs in January, not now

Migration cannot begin before early 2027 in any case.

A new product reaches general availability in December 2026 that would cover both virtual machines
and containers in one platform, which is the combination we need. Today it exists only as a
pre-release, so committing to something else now carries a real risk of migrating twice. Moving 230
machines a second time is the most expensive outcome available to us, by a wide margin.

The measurements this decision depends on, server capacity and storage utilisation, are also
perishable. Anything gathered in September is out of date by January. Collecting them close to the
start is not a delay, it is the correct order of work.

Waiting is only sensible if the intervening months are used. The plan below does that.

## What we do between now and January

This work is needed whichever platform is chosen and none of it is wasted:

- Reconcile the full machine inventory and decide what happens to roughly twenty powered-off machines
that still consume storage
- Design the fly.io failover properly. This is an application problem, not a server problem: which
services are protected, how quickly they must come back, how their data is replicated, how traffic
is switched, and how we switch back
- Correct a fault in the existing high-availability design, described below
- Complete the commercial assessment, handled separately
- Confirm the contract timeline, which determines how much room January has
- Look at the new product's pre-release in October, which costs us very little

## A fault in the current design, being fixed regardless

The existing plan places its management servers across our two locations in a way that does not
survive losing either one. The arrangement looks balanced but is not, and a failure of the larger
half would take the management layer down.

The fix does not need a third site. Instead of one system stretched across both locations, we run a
complete system in each location, with the applications replicated between them. If a location is
lost, that system is lost with it and the other continues. This is simpler, removes a dependency on
buying presence in a third location, and leaves the surviving side at full strength rather than
degraded.

It is worth being clear about what this costs and what it does not solve. Running two complete copies
needs roughly two-thirds more hardware than running one. The leaner alternative is a single system
with an accepted manual recovery taking hours rather than minutes. That is a genuine choice about how
much an outage matters to us, and it is one of the decisions requested below.

It also does not solve the case where both locations are running but the link between them fails. Both
sides would then believe they should be serving. Preventing conflicting writes in that situation is
application-level work and is included in the failover design above.

## Options at a glance

| Option | Free to run | Fits our hardware | Maturity | Note |
| --- | --- | --- | --- | --- |
| Established open-source hypervisor, leading candidate | Yes | Yes | Mature, with a purpose-built import tool for moving our machines off VMware | Closest match to how we already run storage |
| Alternative open-source hypervisor | Yes | Yes | Mature | Second option, one technical limit still to check |
| Container-native platform with virtualization added | Yes | Yes | Proven for containers, less so for our kind of virtual machines | We would build more of it ourselves |
| The originally chosen platform | Yes | No | Mature | Below the vendor's disk minimum; unsupported configuration |
| Commercial enterprise platform | No | Yes | Mature | Paid licence required; excluded on that basis |
| New combined product | Yes | Yes | Ships December 2026, unproven | The reason to decide in January |

## Risks

- The new product's December date could slip. We need a cut-off after which we proceed without it
- Consolidating the production estate onto fewer servers has to be validated against peak load, not
the average. That measurement is outstanding
- Storage headroom is measured as free space reported by the system, which is not the same as
verified physical capacity on the array. To be confirmed
- Running without a support contract is a policy position, not just a default. If we take it, we
should take it deliberately
