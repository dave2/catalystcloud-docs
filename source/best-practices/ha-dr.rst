.. _ha_dr_concepts:

***************************************
High Availability and Disaster Recovery
***************************************

==================
What is HA and DR?
==================

High Availability (HA) and Disaster Recovery (DR) are related but different
concepts. They address two different requirements for a system:

* Ensuring that the system remains available despite failure without human
  intervention; and
* Ensuring data and configuration is recoverable from loss or corruption.

Importantly, these are not interchangeable concepts. High Availability is
*not* a substitute for Disaster Recovery. Similarly, while DR ensures that
data is protected to a known degree, DR processes tend to result in
lengthy downtime, and are not a substitute for High Availability.

Whether a given system needs HA and/or DR is dependant on the tolerance
for the system not being available and the importance of keeping the data
or configuration stored.

.. note::

  Catalyst Cloud can help you identify appropriate mechanisms to implement
  HA and DR practices for your application. Our platform provides a number
  of tools that can simplify and automate both HA and DR practices.

====================
RTO and RPO Concepts
====================

What is RTO and RPO?
====================

The acceptable limit for a system being unavailable, or "downtime",  is
called the *Recovery Time Objective* (RTO) while the acceptable amount of
data that could be lost from any event (even if it does not result in
downtime) is called the *Recovery Point Objective* (RPO). Both of these
numbers are for a single event, and strongly influence the design of systems
and the patterns used.

They are not to be confused with a Service Level Agreement, which will also
specify downtime and data loss limits. Service Level Agreements usually
describe these over long durations (eg, 99.999% availability over a year),
or only the likely occurrence of data loss (eg, 99.9999999% durability). An
SLA typically does not specify how often events may occur, or the limit for
a single event, or how much data may lost in a single event.

The relationship between the RTO and RPO is shown below:

.. aafig::

  Backup  o---------------------------\
          ^                           | Data Restoration
          |<--- RPO --->|             |
          |             C             V
  Live -o-o-o-o-o-o-o-o-C             o-o-o-
                        C              ^
                        ^<--- RTO ---->|
                        |              \- Service Restoration
                        \- Event

When a failure occurs, the maximum time between the start of the failure and
the system being available again is our RTO. If the failure loses data,
the age of the most recent backup we must roll back to is our RPO. A
failure might not lose data, but the maximum time it was down is still
limited to the RTO.

What is 'downtime'?
===================

Although this may seem obvious, it is not always well understood what is
meant by a system being 'down'. This is because we traditionally apply a
system lens to the problem - the server is no longer running, or the
database is not responding - rather than a business lens.

Downtime for systems could be, for some business processes, that they cannot
add new data to a system, even if they can continue to access existing data
in a system. Conversely, it may be that for some processes, the business can
accept a read-only state for some time, which may not require treating the
system as unavailable.

It could also be that the system is unable to process within a reasonable
time frame, for example the database is slow at reading or writing records.

Every system should have a definition of what the business stakeholders
consider to be an 'available' and 'unavailable' system.

Determining RTO and RPO
=======================

Each organisation will have their own way of determining what the RTO and
RPO is for a system. Some common considerations include:

* Financial and Reputation impacts, such as lost revenue, additional staff
  hours to manage the failure, or customer experience
* Whether the system is a master of some data, or is merely a view or
  cache of other systems
* The interdependence between systems and processes, such as whether this
  system provides a critical service to another system

Another common pattern is to group the systems or applications according
to a simple scale, often from 1 to 5 of importance, and then defining
RTO and RPO based on importance.

Using importance, however, can lead to a trap that the business owners
believe if their system classed as the highest importance it will be first,
or even early, in any recovery effort. It is also common to not fully
appreciate that under very extreme circumstances (such as the loss of an
entire location), the RTO tends to be meaningless as there may be simply too
much that needs to be done to restore systems in a large-scale outage.

Importance also hides the added complexity that systems which depend on
other systems introduces. It is unfortunately common to build a system
that has, say, importance of 1 and the lowest RTO while that system
depends on something that is, say, importance 3 and has a very relaxed
RTO. The best practice is to "roll up" the longest RTO and RPO of any
dependencies as the actual RTO and RPO of a given system.

Engagement with business stakeholders to define RTO and RPO is critical
to ensure both that the business stakeholders understand what these
limits mean, and that technical teams understand the requirements they
need to design for.

.. note::

  Although Catalyst Cloud cannot determine the right RTO or RPO value for
  your organisation, we can provide guidance about the likely ability for
  a design to meet a known RTO or RPO, and facilitate discussions with
  stakeholders on industry-common RTO and RPO values.

===============
Failure Domains
===============

The assurance provided by an HA or DR design stems from knowledge of the
*failure domains* that components share between each other, and the
system as a whole. Failure domains can be thought of as a ring around
components that will cause them to fail as a group in the face of some
event.

Catalyst Cloud operates your resources in different failure domains
based on the type of resource and the way you specify any placement
or constraints that need to be honoured.

.. aafig::

  +-----------------------+ +-----------------------+
  | "Region A"            | | "Region B"            |
  | +--------+ +--------+ | | +--------+ +--------+ |
  | | "AZ-A" | | "AZ-B" | | | | "AZ-A" | | "AZ-B" | |
  | | +----+ | | +----+ | | | | +----+ | | +----+ | |
  | | | HW | | | | HW | | | | | | HW | | | | HW | | |
  | | +----+ | | +----+ | | | | +----+ | | +----+ | |
  | |        | |        | | | |        | |        | |
  | | +----+ | | +----+ | | | | +----+ | | +----+ | |
  | | | HW | | | | HW | | | | | | HW | | | | HW | | |
  | | +----+ | | +----+ | | | | +----+ | | +----+ | |
  | +--------+ +--------+ | | +--------+ +--------+ |
  |                       | |                       |
  | +-------------------+ | | +-------------------+ |
  | | Control Plane     | | | | Control Plane     | |
  | +-------------------+ | | +-------------------+ |
  |                       | |                       |
  | +-------------------+ | | +-------------------+ |
  | | Regional Services | | | | Regional Services | |
  | +-------------------+ | | +-------------------+ |
  +-----------------------+ +-----------------------+

  +-------------------------------------------------+
  | Global Services and Control Plane               |
  +-------------------------------------------------+

There are broadly four failure domains that could affect a resource:

* Resources which share the same hardware are sharing a failure domain,
  specifically that piece of hardware
* Resources created within an Availability Zone (or "AZ") share building
  level facilities (such as power or cooling), access to the location,
  and exposure to some natural disasters such as flooding. They may
  also share a control plane or configuration with components that
  make up the resource within an AZ
* Resources created within a Region share a control plane across all
  resources in the region, and may share national or interational
  network links
* Global resources share a control plane for the resource everywhere,
  regardless of location

Every failure domain has certain risks and events that are likely to
affect it. For example, at the level of sharing hardware these are
simple events such as cabling being dislodged, power supplies within
the hardware, or bad RAM. Conversely, at the region level the risks
are largely about events that would affect a whole city such as unrest
or a volcanic eruption.

.. note::

  Although each of these levels is a failure domain, there is redundancy
  built into every one of these layers to avoid common causes of failure.
  For example, every server has redundant power supplies and power
  connections, the region control plane is distributed and fault tolerant
  over multiple AZs, and global services are implemented in a way that
  ensures disconnection between regions does not affect the global service.

As part of your design to ensure you can meet the expected RTO and RPO, you
must take into account these failure domains for any resource.

In Catalyst Cloud, you can ensure that your resources do not share a
failure domain by:

* Placing servers in an anti-affinity group to ensure they do not run on
  the same hardware
* Place resources like servers and block storage in different Availability
  Zones
* Making copies of data placed in different AZs or Regions
* Create complete copies of an application stack in different Regions

=================
High Availability
=================

High Availability is intended to avoid failures making a system unavailable.
However, they are not an alternative to Disaster Recovery, and every system
will still need a DR strategy even if it uses HA techniques.

Most implementations of HA do not completely remove all impacts of failures.
There is usually some delay between a failure and the HA design handling
the failure. Since handling failure is automated in a HA design, the
duration of impact may be greatly reduced, but it will not be zero.

Further, although HA may reduce the impact of some failures, there are other
failures that HA will be unable to handle. This means that your worst case
RTO should not be based on the time for HA to recover a fully functional
system, but on the time to complete Disaster Recovery processes.

Availability Patterns
=====================

HA patterns can be broadly broken into two classes:

* Active-Passive
* Active-Active

In Active-Passive there is an active resource and a passive resource. The
active resource receives all traffic or runs all processes, while
the passive resource remains idle, except for state synchronisation.

When the active resource fails, the passive resource is promoted and takes
over all traffic or processing. The active side is effectively treated as
dead. For the passive side to take over it needs access to state as
recently as possible, and no further back than the RPO of the system.

In Active-Active all instances of resources are able to be used and receive
traffic or run processes, and operate simultaneously. Active-Active
typically imposes stricter requirements on state synchronisation and data
storage, as well as the need to manage concurrency (as the resources are
running in parallel).

This means that for some layers of a stack it may be easiest to implement
Active-Active, while other layers are easier as Active-Passive.
Understanding the pattern for each layer is very important to understanding
how the stack or application as a whole responds to different types
of failures.

.. note::

  Active-Active is not 'better' than Active-Passive. Active-Active may
  reduce the impact of some failures, while at the same time introducing
  new types of failures to the whole system (most commonly data corruption
  when concurrency is not managed carefully). Active-Passive with automatic
  fail over can often be an acceptable compromise between completely manual
  failure handling and Active-Active designs.

Resource Isolation
==================

High Availability depends largely on keeping resources isolated from
each other, that is putting them in different failure domains as much
as possible.

For example, putting two servers in the same AZ is not as highly available
as in different AZs, even if the two servers are in an anti-affinity group.

Understanding the failure domains as outlined above is critical to knowing
how much the system is actually highly available.

=================
Disaster Recovery
=================

While High Availability attempts to hide the consequences of failures,
Disaster Recovery (DR) is all about dealing with consequences that result in
the need to wipe whatever condition the system is in now and rebuild it to a
past, known-good, state.

The triggering of DR is generally an explicit, human-driven decision and
assumes that there is nothing of value that can be recovered from the
current state of the system.

The core to any DR strategy is to have a copy of all artefacts needed to
rebuild the system to a past point in time available and protected from
failures that would affect the running system. Artefacts include not
only data but configuration and infrastructure layout.

In addition, any DR strategy that has not been regularly tested is almost
certainly not safe to rely on. Testing that your DR process works and
results in the system being returned to the point you expect it to be at
is a mandatory part of having a DR strategy.

Principles of DR
================

Disaster Recovery has a number of traps to avoid when considering how it
should be implemented for any system. There are a few principles that
should be kept in mind when defining the DR approach for any system.

* Systems always need a DR strategy
* Copies of any data or configuration should be independent
* Copies should not share the same failure domains as the source systems
* HA is not DR

It is a common phrase to hear that a system "doesn't need DR", but this is
very rarely the case. Every system should have a means to be recovered,
but recovery does not necessarily mean "backups".

Other recovery options could include:

* Recreating the systems from infrastructure-as-code configuration and tools
* Replaying or recreating data from another, protected, system
* Starting from an empty data set (eg, accepting the loss of cached data)

To ensure that any of these strategies are able to be executed, at least
some information is protected. Protecting data is deeply rooted in the
concept that the protected copy is not affected by what happens with the
live or other copied data. There are two ways a copy could be affected:
losing a link in a chain of copies, or sharing a failure domain.

When making a copy, it is attractive to reduce the amount of data being
copied by creating an "incremental" copy - that is, storing only the
differences between the current state and the last copy. However, if
the first full copy is lost, the incremental copies are effectively
useless. Only full copies are actually independent from each other.

The other case is where any copy or live data share a failure domain,  care
must be taken to ensure that the possible failures are understood. For
example, placing backups on the same storage platform as the live data will
definitely share a failure domain, which makes the reliability of the
backups greatly reduced. However, placing a copy of data in the same AZ as
the live data may be acceptable if they different storage platforms,
depending on how the data is distributed across the AZ. It also may be
acceptable to place multiple full copies of the data in the same storage
system, depending on how well the storage system ensures the data can be
recovered.

.. note::

  Catalyst Cloud provides two independent storage systems: block, and
  object. Block is suitable for "live" data, and object is automatically
  duplicated across either an AZ or a whole region depending on the
  way you have configured the object container. Consult the documentation
  for our Block and Object systems for further information on their
  performance, durability, and use cases for DR.

Lastly, as noted above, it may not be the system itself which is protected
but the scripts and configuration which creates it. In this case, the
same principles apply to the system which stores the scripts and
configuration needed to re-create the system, as well as any access required
to execute re-creating the system.

=====================
Multi-Region Patterns
=====================

A common requirement to reduce the time to recover data, and avoid
situations of loss as much as possible, is to establish multiple copies
of the system and all data in diverse locations, usually different
regions. These patterns may also be required to scale a system to
ensure it is closer to the consumer than a single region would allow, and
may satisfy some HA requirements.

Multiple regions imposes some complexities because it is not common for
regions to share a control plane - there are few resources which are
truly global. There are also complexities about directing traffic into
different regions, stemming from network design.

Multi-region design falls into the following patterns:

* Pilot Light, running very few systems but maintaining a complete copy
  of scripts, tools, and data to recover the system
* Active-Passive, where some or all resources are running on a passive
  region and this region can be activated by redirecting incoming traffic
* Active-Active, where each region is fully active and every region is
  accepting traffic

Each of these levels adds complexity and cost to the implementation.

Inbound Traffic Steering
========================

All three of the patterns described above require some method of controlling
which region traffic is being directed to, whether automatically or manually
controlled.

There are several different approaches and layers which can be placed into
the inbound traffic path to distribute traffic. Some techniques include:

* DNS changes, such as manually changing what IP address a name resolves to,
  or automatically monitoring where the name points to and failing over
  to another address during any failures
* DNS geolocation, using where the DNS query originated from to pick an
  IP address to pass back that selects a "close" and implicitly "active"
  region
* Proxy policy, where the entry point is a proxy service that performs
  similar needs as the DNS examples above, often provided with DDOS and
  security protection of websites
* Anycast, where a block of IP addresses is advertised to the Internet
  from multiple locations, and global Internet routing policy is used
  to pick a "close" and implicitly "active" site

These approaches are often mixed in to each layer, eg using DNS geolocation
to resolve the closest active proxy performing DDOS protection and routing
to the active region your resources are in.

Pilot Light Multi-Region
========================

A "pilot light" region has a very small number of running services, and
could be thought of as "cold" for your application, although the
infrastructure that underpins the pilot light is usually fully capable and
running.

They are cheap to operate as the costs are mostly data storage and not
running systems. However, a pilot light inherently takes longer to activate
than a warmer system and it runs the risk that not everything needed to
turn the pilot light into a full site is actually available.

They are a good choice for a "site of last resort", but less good as the
first line of responding to a large failure. They are easy to implement,
as they just need to have data and configuration copied into a cold data
store (eg, object storage) and tested from time to time.

Pilot light tends to go hand-in-hand with manual DNS changes, just due to
the manual nature of activating the site.

Active-Passive Multi-Region
===========================

Active-Passive over multiple regions has your application stack running
in two or more regions, but only one has traffic steered towards it and
the others are all slaves from the active region for any data or
configuration.

The benefit of Active-Passive over pilot light is the time needed for
failover, and that failure detection can drive an automatic failover between
the regions. Automation failover, however, does depend on ensuring there
are barriers ensuring only one site is authoratitive for data and
configuration.

Aspects to consider for this pattern include:

* Ensuring data and configuration is replicated between the regions. For
  infrastructure, using templates and orchestration is critical to
  maintaining synchronisation.

* Fencing off broken regions, when they fail you want to ensure any method
  of passing data between the broken region and other regions is disabled.
  Often, this means the remaining running regions need to be explicitly
  told to disregard the broken region.

* Fail back strategies, which will at a minimum involve resyncing the old
  active region to current state and a manually-triggered failover.

A strong point of this pattern is avoiding the need for multi-master data
management, which can be difficult to design and implement in a reliable
manner.

.. note::

  All regions should themselves be highly available, so that the cases
  where failover between regions is required is minimised.

Active-Active Multi-Region
==========================

Running an application stack with multiple active regions shares many
problems with running active-active HA in a single region. The problems
largely stem from managing state and data consistency. If you don't need
to manage state or data, then multi-region is much easier to implement.

It is rarely the case that regions are close enough to allow any sort of
synchronous behaviour between regions, but this is always a design choice
available. Generally asynchronous methods are employed.

Data and state replication is easiest with the *eventually consistent*
model, an asynchronous pattern where we accept that regions each have a
lagged view of the world but do not introduce conflicts with each other. For
example, each region could write to a replicated message stream about the
actions they are taking, which the other regions will read and apply as they
see the messages. Each region would need to generate guaranteed unique
identifiers for any new changes and must not assume orderedness (because
parts of a flow of steps could hit different regions).

As all of this is highly dependant on the application involved, it is
beyond the scope of this documentation to explore these ideas.

Lastly, traffic steering should take into account the responsiveness of the
different regions (ie, both latency to the locations and the capacity of the
region).

.. note::

  Catalyst Cloud can provide Professional Services that can help you
  address multi-region patterns and practices for your applicaitons.
