---
layout: "docs"
page_title: "Frequently Asked Questions"
sidebar_current: "docs-faq"
---

# Frequently Asked Questions

## Q: Why is virtual memory usage high?

Consul makes use of [LMDB](http://symas.com/mdb/) internally for various data
storage purposes. LMDB relies on using memory-mapping, a technique in which
a sparse file is represented as a contiguous range of memory. Consul configures
high limits for these file sizes and as a result relies on large chunks of
virtual memory to be allocated. However, in practice, the limits are much larger
than any realistic deployment of Consul would ever use, and the resident memory or
physical memory used is much lower.

## Q: What is Checkpoint? / Does Consul call home?

Consul makes use of a HashiCorp service called [Checkpoint](http://checkpoint.hashicorp.com)
which is used to check for updates and critical security bulletins.
Only anonymous information, which cannot be used to identify the user or host, is
sent to Checkpoint . An anonymous ID is sent which helps de-duplicate warning messages.
This anonymous ID can can be disabled. In fact, using the Checkpoint service is optional
and can be disabled.

See [`disable_anonymous_signature`](/docs/agent/options.html#disable_anonymous_signature)
and [`disable_update_check`](/docs/agent/options.html#disable_update_check).

## Q: How does Atlas integration work?

Consul makes use of a HashiCorp service called [SCADA](http://scada.hashicorp.com)
(Supervisory Control And Data Acquisition). The SCADA system allows clients to maintain
long-running connections to Atlas. Atlas can in turn provide auto-join facilities for
Consul agents (supervisory control) and an integrated dashboard showing the health of
all connected agents (data acquisition).

Standard ACLs can be applied to the SCADA connection, ensuring that Atlas is given only
those privileges that make sense for your deployment.

Using the SCADA service is optional. SCADA is only enabled by opt-in.

See the [Atlas integration guide](/docs/guides/atlas.html) for more details.

## Q: Does Consul rely on UDP Broadcast or Multicast?

Consul uses the [Serf](https://serfdom.io) gossip protocol which relies on
TCP and UDP unicast. Broadcast and Multicast are rarely available in a multi-tenant
or cloud network environment. For that reason, Consul and Serf were both
designed to avoid any dependence on those capabilities.

## Q: Is Consul eventually or strongly consistent?

Consul has two important subsystems, the service catalog and the gossip protocol.
The service catalog stores all the nodes, service instances, health check data,
ACLs, and Key/Value information. It is strongly consistent, and replicated
using the [consensus protocol](/docs/internals/consensus.html).

The [gossip protocol](/docs/internals/gossip.html) is used to track which
nodes are part of the cluster and to detect a node or agent failure. This information
is eventually consistent by nature. When the servers detects a change in membership,
or receive a health update, they update the service catalog appropriately.

Because of this split, the answer to the question is subtle. Almost all client APIs
interact with the service catalog and are strongly consistent. Updates to the
catalog may come via the gossip protocol which is eventually consistent, meaning
the current state of the catalog can lag behind until the state is reconciled.

